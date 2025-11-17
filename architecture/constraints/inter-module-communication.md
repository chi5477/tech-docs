# 模組間通訊（同步/非同步）

> 原則：同步路徑處理超時與錯誤邊界；非同步路徑具備冪等、去重與可觀測性。

---

## 設計取捨
- 同步（RPC/REST/函式呼叫）：強一致、即時錯誤；注意延遲、重試、超時與回退。
- 非同步（事件/訊息）：時間去耦、彈性伸縮；需處理最終一致、去重、重送與追蹤。

---

## Go 範例：同步呼叫（用例依賴介面）
```go
// 檔案：src/application/usecase/charge_order.go
package usecase

import (
    "context"
    "time"
    "example.com/project/src/application/ports"
)

type ChargeOrder struct { payments ports.PaymentProvider }

func NewChargeOrder(p ports.PaymentProvider) *ChargeOrder { return &ChargeOrder{payments: p} }

func (uc *ChargeOrder) Exec(ctx context.Context, orderID string) (string, error) {
    ctx, cancel := context.WithTimeout(ctx, 3*time.Second)
    defer cancel()
    id, err := uc.payments.Charge(ctx, 100, "USD", orderID)
    if err != nil { return "", err }
    return string(id), nil
}
```

---

## Go 範例：非同步事件總線（InMemory Adapter）
```go
// 檔案：src/application/ports/event_bus.go
package ports

import "context"

type Event interface { Name() string }

type EventBus interface {
    Publish(ctx context.Context, e Event) error
    Subscribe(name string, h func(ctx context.Context, e Event)) (unsubscribe func())
}
```

```go
// 檔案：src/adapters/bus/inmem/bus.go
package inmem

import (
    "context"
    "sync"
    "example.com/project/src/application/ports"
)

type Bus struct {
    mu   sync.RWMutex
    subs map[string][]func(context.Context, ports.Event)
}

func New() *Bus { return &Bus{subs: map[string][]func(context.Context, ports.Event){}} }

func (b *Bus) Publish(ctx context.Context, e ports.Event) error {
    b.mu.RLock(); hs := append([]func(context.Context, ports.Event){}, b.subs[e.Name()]...); b.mu.RUnlock()
    for _, h := range hs { h(ctx, e) }
    return nil
}

func (b *Bus) Subscribe(name string, h func(context.Context, ports.Event)) (func()) {
    b.mu.Lock(); b.subs[name] = append(b.subs[name], h); b.mu.Unlock()
    return func() { b.mu.Lock(); defer b.mu.Unlock(); b.subs[name] = remove(b.subs[name], h) }
}

func remove(x []func(context.Context, ports.Event), t func(context.Context, ports.Event)) []func(context.Context, ports.Event) {
    for i := range x { if &x[i] == &t { return append(x[:i], x[i+1:]...) } }
    return x
}

var _ ports.EventBus = (*Bus)(nil)
```

```go
// 檔案：src/domain/events/order_paid.go
package events

type OrderPaid struct{ OrderID string }
func (OrderPaid) Name() string { return "domain.order.paid" }
```

---

## 可靠性與契約
- 冪等與去重：事件包含唯一鍵（如 AggregateID+版本）；處理端可重入。
- 觀測性：在 Adapter 層加入追蹤/記錄，保留事件 ID、關聯 ID。
- 合約測試：對 `EventBus` 的 Publish/Subscribe 與回呼語意撰寫契約測試。

---

## 檢核表
- [ ] 同步路徑具超時、重試與回退策略。
- [ ] 非同步路徑具冪等鍵與去重機制。
- [ ] 有效追蹤（trace/span）與記錄事件關聯 ID。

