# 介面設計與替換性

> 原則：小而專注、可替換（LSP）、契約明確；避免洩漏第三方型別。

---

## 設計準則
- 介面分割：以使用者（呼叫方）需求為中心，避免上帝介面。
- 符合 LSP：不同實作在同一契約下可互換（輸入/輸出/錯誤語意一致）。
- 背景通用性：接受 `context.Context`、回傳 `error` 描述取消與超時。
- 契約測試：以合約測試約束行為，避免實作分歧。

---

## Go 範例：支付介面與兩種實作
```go
// 檔案：src/application/ports/payment_provider.go
package ports

import "context"

type ChargeID string

type PaymentProvider interface {
    Charge(ctx context.Context, amount int64, currency, ref string) (ChargeID, error)
}
```

```go
// 檔案：src/adapters/outbound/stripe/stripe.go
package stripe

import (
    "context"
    "fmt"
    "example.com/project/src/application/ports"
)

type Client struct{ apiKey string }

func New(apiKey string) *Client { return &Client{apiKey: apiKey} }

// 編譯期檢查符合介面
var _ ports.PaymentProvider = (*Client)(nil)

func (c *Client) Charge(ctx context.Context, amount int64, currency, ref string) (ports.ChargeID, error) {
    return ports.ChargeID(fmt.Sprintf("tx_%s", ref)), nil
}
```

```go
// 檔案：src/adapters/outbound/mockpay/mockpay.go
package mockpay

import (
    "context"
    "example.com/project/src/application/ports"
)

// 編譯期檢查符合介面
var _ ports.PaymentProvider = (*Mock)(nil)

type Mock struct{ Err error }

func (m *Mock) Charge(ctx context.Context, amount int64, currency, ref string) (ports.ChargeID, error) {
    if m.Err != nil { return "", m.Err }
    return ports.ChargeID("tx_mock_" + ref), nil
}
```

---

## 設計建議
- 自定義錯誤並以 `errors.Is/As` 判斷重試/使用者/系統錯誤。
- 不回傳第三方 SDK 型別，必要時以自家型別包裝。
- 將退款、查詢等職責以獨立介面拆分（`RefundProvider` 等）。

---

## 檢核表
- [ ] 介面小而專注，符合呼叫方需求。
- [ ] 編譯期替換性檢查通過（`var _ IFace = (*Impl)(nil)`）。
- [ ] 錯誤語意一致且具合約測試。

