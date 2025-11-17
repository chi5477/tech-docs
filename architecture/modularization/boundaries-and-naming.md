# 模組邊界、命名與耦合（含 Go 範例）

> 本文件聚焦「功能模組化」下，如何畫清邊界、取好名字並控管耦合。若只有把檔案分類成功能模組，沒有最小的架構規則，實務上仍會出現牽一髮動全身、重複實作與可讀性下降等問題。以下提供可直接落地的準則與 Go 範例。

---

## 模組邊界判準（功能導向）
- 業務能力：以「完成一件業務價值」為單位（`orders`、`billing`）。
- 變動速率：高變動與低變動分離，避免小改動牽動多模組。
- 一致性需求：需要強一致的行為聚合在同一模組/聚合內。
- 協作關係：跨模組僅透過介面/事件互動，避免讀寫彼此內部型別或資料表。
- 擁有權：資料與決策需有明確 Owner；他模組只能以查詢或事件取得資訊。

反例提示：
- 用技術分層命名資料夾（`controllers/services/repositories`）卻把業務邏輯散落各處，導致橫切耦合與「共用大泥球」。
- 多模組共用同一資料表或 ORM 實體，資料所有權不清，改動風險高。

---

## Go 範例：以 Ports 隔離 SQL，防止牽一髮動全身
```go
// 檔案：src/modules/orders/application/ports/order_repo.go
package ports

import (
    "context"
    "example.com/project/src/modules/orders/domain"
)

// 由應用層定義倉儲 Port，禁止直接在用例中寫 SQL
type OrderRepo interface {
    Save(ctx context.Context, o *domain.Order) error
    Find(ctx context.Context, id domain.OrderID) (*domain.Order, error)
}
```

```go
// 檔案：src/modules/orders/infrastructure/repo/sql/order_repo_sql.go
package sqlrepo

import (
    "context"
    "database/sql"
    "time"
    "example.com/project/src/modules/orders/application/ports"
    "example.com/project/src/modules/orders/domain"
)

// 未匯出實作，對外僅以建構函式回傳介面，避免外部依賴具體型別
type orderRepoSQL struct { db *sql.DB }

func NewOrderRepo(db *sql.DB) ports.OrderRepo { return &orderRepoSQL{db: db} }

func (r *orderRepoSQL) Save(ctx context.Context, o *domain.Order) error {
    _, err := r.db.ExecContext(ctx,
        "INSERT INTO orders(id, paid, created_at) VALUES ($1,$2,$3) ON CONFLICT (id) DO UPDATE SET paid=$2",
        o.ID(), o.IsPaid(), time.Now())
    return err
}

func (r *orderRepoSQL) Find(ctx context.Context, id domain.OrderID) (*domain.Order, error) {
    return domain.NewOrder(id, time.Now())
}
```

```go
// 檔案：src/modules/orders/application/usecase/charge_order.go
package usecase

import (
    "context"
    "example.com/project/src/modules/orders/application/ports"
)

type ChargeOrder struct { repo ports.OrderRepo }

func NewChargeOrder(r ports.OrderRepo) *ChargeOrder { return &ChargeOrder{repo: r} }

func (uc *ChargeOrder) Exec(ctx context.Context, id string) error {
    o, err := uc.repo.Find(ctx, ports.OrderID(id)) // 依賴介面，不寫 SQL
    if err != nil { return err }
    o.MarkPaid()
    return uc.repo.Save(ctx, o)
}
```

重點：SQL 只存在於 `infrastructure/repo`，如果查詢策略或 schema 調整，改一處 Adapter 即可，不必在各個用例或模組中全面搜尋修改。

---

## 命名準則（含 Go 慣例）
- 模組名以業務語彙命名：`orders`, `billing`, `catalog`，避免以 `utils`、`common` 作為功能模組名。
- 穩定字彙：建立名詞表（參見 `docs/architecture/glossary.md`），跨模組名稱一致。
- Go 套件命名：小寫、短且可讀，不使用底線與複數變化混雜（如 `sqlrepo`, `usecase`）。
- 介面/Port 命名：以能力為名，不加 `I` 前綴；行為導向可用 `-er`（`Notifier`, `Publisher`）。
- 版本與契約：對外 API/事件採語意化版本（如 `v1`），並在對應文件註記。

---

## 耦合管理與跨模組協作
- 依賴反轉：跨模組以介面（Ports/Interfaces）耦合；實作於 Adapter 層。
- 資料形狀隔離：跨模組以 DTO/事件載荷互通，避免共享內部領域型別。
- 同步/非同步選擇：
  - 同步（函式/REST/gRPC）：即時回饋，但有時間耦合；需設置超時與重試政策。
  - 非同步（事件/訊息）：時間去耦，利於擴展；需處理最終一致與去重。
- 防腐層（ACL）：對接外部或異質模組，使用 ACL 避免外部模型污染核心。

Go 範例：跨模組以 DTO 互通（避免共享內部型別）
```go
// 檔案：src/modules/billing/application/dto/order_snapshot.go
package dto

type OrderSnapshot struct {
    ID    string `json:"id"`
    Paid  bool   `json:"paid"`
}
```

```go
// 檔案：src/modules/billing/application/ports/order_query.go
package ports

import (
    "context"
    "example.com/project/src/modules/billing/application/dto"
)

type OrderQuery interface {
    GetSnapshot(ctx context.Context, id string) (*dto.OrderSnapshot, error)
}
```

```go
// 檔案：src/modules/orders/interfaces/query/order_query_impl.go
package query

import (
    "context"
    "example.com/project/src/modules/billing/application/dto"
    "example.com/project/src/modules/orders/application/ports"
)

type orderQueryImpl struct { repo ports.OrderRepo }

func NewOrderQuery(repo ports.OrderRepo) interface{ GetSnapshot(context.Context,string)(*dto.OrderSnapshot,error) } {
    return &orderQueryImpl{repo: repo}
}

func (q *orderQueryImpl) GetSnapshot(ctx context.Context, id string) (*dto.OrderSnapshot, error) {
    o, err := q.repo.Find(ctx, ports.OrderID(id))
    if err != nil { return nil, err }
    return &dto.OrderSnapshot{ID: string(o.ID()), Paid: o.IsPaid()}, nil
}
```

---

## 反模式（Anti-patterns）與 Go 片段
- 跨模組直接依賴他方 Repository/實作
```go
// 🚫 反例：billing 直接 import orders 的 SQL 實作，造成耦合與方向反轉
import "example.com/project/src/modules/orders/infrastructure/repo/sql" // 禁止
```

- 共享內部領域型別作為跨模組協定
```go
// 🚫 反例：對外回傳 orders.domain.Order，外部可任意存取內部狀態
type Order struct { ID string; Paid bool } // 被外部直接依賴
```

- 全域單例濫用
```go
// 🚫 反例：以全域變數暴露 DB 實例，任何模組都能直接使用
var DB *sql.DB
```

- import cycle / 方向反轉
```go
// 🚫 反例：domain 匯入 adapters 或其他外層模組
// import "example.com/project/src/modules/orders/infrastructure/repo/sql" // 禁止
```

改正方式：對外只暴露介面/DTO/事件；實作放在 Adapter；在組合根注入。

---

## 目錄建議（功能模組化 + 最小架構）
```
src/
  modules/
    orders/
      domain/
      application/
        ports/
        usecase/
      interfaces/
      infrastructure/
        repo/
        gateway/
    billing/
      ...
  shared/
    kernel/
    acl/
  app/
    di/
    config/
```

---

## 檢核表
- [ ] 模組名稱清楚表達業務能力，避免技術導向命名。
- [ ] 跨模組互動未使用內部型別或資料表，僅透過介面/DTO/事件。
- [ ] SQL/HTTP 只存在於 Adapter 層，核心層不含外部 I/O 細節。
- [ ] 無 import 方向反轉或循環，實作不被外部直接引用（`internal/`、未匯出）。
- [ ] 公開 API/事件具備版本化與文件；測試覆蓋跨模組契約。
