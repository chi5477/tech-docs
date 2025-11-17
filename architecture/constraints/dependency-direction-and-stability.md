# 依賴方向與穩定性

> 原則：依賴只向內；避免循環；讓穩定的模組承載抽象（Stable Abstractions）。

---

## 核心規則
- 外層僅依賴內層：Interfaces/Infrastructure/Adapters → Application → Domain。
- 無循環依賴：套件（package）與模組之間不得形成 import cycle。
- 穩定抽象原則：越穩定（被更多模組依賴）的模組，越應由抽象（介面）組成。
- 開放封閉：對擴充開放、對修改封閉；以新增 Adapter/實作取代修改核心。

---

## Go 組織方式（建議）
```
src/
  domain/                 # 實體、值物件、領域服務
  application/
    ports/                # 介面（Ports）
    usecase/              # 用例
  adapters/
    inbound/              # web/grpc/cli/scheduler
    outbound/             # db/cache/message/3rd-party
  app/
    di/                   # 組合根
```

關係：`adapters -> application -> domain`，禁止 `domain -> application` 或 `domain -> adapters`。

---

## Go 範例：導向內層的匯入
```go
// 檔案：src/adapters/repo/sql/order_repo_sql.go
package sqlrepo

import (
    "example.com/project/src/application/ports" // 向內依賴介面
    "example.com/project/src/domain"           // 讀取公開模型（方法守護狀態）
)

// 依賴內層的抽象進行實作
type orderRepoSQL struct{ /* ... */ }
var _ ports.OrderRepo = (*orderRepoSQL)(nil)
```

反例（禁止）：
```go
// 檔案：src/domain/anti_example.go
package domain

// 反例：domain 依賴 adapters，造成方向反轉與強耦合
// import "example.com/project/src/adapters/repo/sql" // 🚫 禁止
```

---

## 穩定性指標（實務判準）
- 內聚：模組內檔案彼此關聯緊密；變動多發生於同一模組內。
- 出/入向依賴：出向依賴越少、入向依賴越多，模組越穩定。
- 穩定模組應抽象化：介面、DTO、事件契約優先；避免攜帶具體基礎設施型別。

---

## 檢核表
- [ ] import 方向只朝內層（adapters→application→domain）。
- [ ] 無 import cycle；穩定模組以抽象為主。
- [ ] 新增 I/O 能力以新增 Adapter/實作為主，不修改核心。

