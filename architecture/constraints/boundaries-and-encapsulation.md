# 邊界、可見性與封裝

> 原則：核心層不依賴外層；以小寫未匯出與 `internal/` 保護實作；只公開必要 API 與抽象（Ports）。

---

## 原則
- 依賴反轉：介面（Ports）放在 Application/Domain；實作在 Adapters/Infrastructure。
- 隱藏內部：利用小寫未匯出與 `internal/` 目錄避免外部直接耦合細節。
- 穩定 API：避免在核心層曝露第三方或資料存取型別。

---

## Go 範例：防止內部狀態外洩與倉儲 Port
```go
// 檔案：src/domain/order.go
package domain

import (
    "errors"
    "time"
)

type OrderID string

type Order struct { // 未匯出欄位保護不變式
    id        OrderID
    createdAt time.Time
    paid      bool
}

func NewOrder(id OrderID, now time.Time) (*Order, error) {
    if id == "" { return nil, errors.New("empty id") }
    return &Order{id: id, createdAt: now}, nil
}

func (o *Order) ID() OrderID   { return o.id }
func (o *Order) IsPaid() bool  { return o.paid }
func (o *Order) MarkPaid()     { o.paid = true }
```

```go
// 檔案：src/application/ports/order_repo.go
package ports

import (
    "context"
    "example.com/project/src/domain"
)

type OrderRepo interface {
    Save(ctx context.Context, o *domain.Order) error
    Find(ctx context.Context, id domain.OrderID) (*domain.Order, error)
}
```

```go
// 檔案：src/adapters/repo/sql/order_repo_sql.go
package sqlrepo

import (
    "context"
    "database/sql"
    "time"
    "example.com/project/src/application/ports"
    "example.com/project/src/domain"
)

type orderRepoSQL struct { db *sql.DB } // 未匯出型別

func NewOrderRepo(db *sql.DB) ports.OrderRepo { // 只回傳介面
    return &orderRepoSQL{db: db}
}

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

---

## 可見性策略
- 小寫未匯出欄位與方法守護不變式；以公開方法暴露受控操作。
- `internal/` 用於禁止跨模組依賴實作細節。
- 建構函式回傳介面避免外部依賴具體型別。

---

## 檢核表
- [ ] 核心層不曝露第三方或資料儲存型別。
- [ ] 未匯出欄位與 `internal/` 妥善使用。
- [ ] 對外回傳介面、實作置於外層。

