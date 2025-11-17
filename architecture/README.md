# 架構與程式碼結構索引（Index）

> 本索引提供 Clean、Hexagonal、Onion、DDD 與模組化相關文件的導覽地圖與連結規劃。所有文件採用 UTF-8（無 BOM）、繁體中文為使用者溝通語言。

---

## 如何使用本索引
- 先自「快速導覽」進入對應主題或架構頁面（README）。
- 於各主題頁面內可進一步前往子章節（概念、目錄模板、依賴規則、反模式、檢核表、範例）。
- 若尚未建立目標頁面，請依本索引的檔案路徑與命名新增對應 `.md` 文件。

## 快速導覽
- [Clean Architecture](clean/README.md)
- [Hexagonal（Ports & Adapters）](hexagonal/README.md)
- [Onion Architecture](onion/README.md)
- [DDD（Domain-Driven Design）](ddd/README.md)
- [模組化與分層原則](modularization/README.md)
- [依賴規則與邊界](constraints/README.md)
- [測試策略](testing/README.md)
- [文件範本與腳手架](templates/README.md)
- [ADR 索引（Architecture Decision Records）](adr/README.md)
- [名詞表與通用語彙](glossary.md)
- [常見問答（FAQ）](faq.md)

---

## 文件地圖與建議路徑

> 以下列出各主題的建議目錄結構與子文件，做為可點入的資訊架構藍圖。

### Clean Architecture
- 概述與準則：`clean/README.md`
- 目錄模板與骨架：`clean/directory-structure.md`
- Use Cases 與 Entities：`clean/use-cases-and-entities.md`
- Ports（介面）與相依反轉：`clean/ports-and-di.md`
- Controllers/Presenters 指南：`clean/controllers-presenters.md`
- Infrastructure（資料庫/訊息/外部服務）：`clean/infrastructure.md`
- 反模式與常見錯誤：`clean/anti-patterns.md`
- 檢核表（Checklist）：`clean/checklist.md`
- 範例與走查：`clean/examples.md`

### Hexagonal（Ports & Adapters）
- 概述與準則：`hexagonal/README.md`
- 目錄模板與骨架：`hexagonal/directory-structure.md`
- Inbound/Outbound Ports 設計：`hexagonal/ports.md`
- Adapters（Web/CLI/Events/DB）指南：`hexagonal/adapters.md`
- 依賴與測試策略：`hexagonal/dependencies-and-testing.md`
- 反模式與常見錯誤：`hexagonal/anti-patterns.md`
- 檢核表（Checklist）：`hexagonal/checklist.md`
- 範例與走查：`hexagonal/examples.md`

### Onion Architecture
- 概述與準則：`onion/README.md`
- 目錄模板與骨架：`onion/directory-structure.md`
- 同心層（Domain→Application→Infrastructure）：`onion/layers.md`
- Repository/Specification 等模式：`onion/patterns.md`
- 依賴與封裝規則：`onion/dependencies-and-encapsulation.md`
- 反模式與常見錯誤：`onion/anti-patterns.md`
- 檢核表（Checklist）：`onion/checklist.md`
- 範例與走查：`onion/examples.md`

### DDD（Domain-Driven Design）
- 概述與準則：`ddd/README.md`
- 有界上下文與 Context Mapping：`ddd/bounded-contexts-and-mapping.md`
- 聚合、實體、值物件：`ddd/aggregates-entities-value-objects.md`
- 領域事件與整合：`ddd/domain-events-and-integration.md`
- Repository 與應用服務：`ddd/repositories-and-application-services.md`
- 防腐層（ACL）與跨 Context 協作：`ddd/anti-corruption-layer.md`
- 反模式與常見錯誤：`ddd/anti-patterns.md`
- 檢核表（Checklist）：`ddd/checklist.md`
- 範例與走查：`ddd/examples.md`

### 模組化與分層原則
- 概述與目標：`modularization/README.md`
- 模組邊界、命名與耦合：`modularization/boundaries-and-naming.md`
- 相依注入與組合根（Composition Root）：`modularization/composition-root.md`
- 介面優先與契約（Contracts/Versioning）：`modularization/contracts-and-versioning.md`
- 事件、訊息總線與整合風格：`modularization/events-and-bus.md`
- 共用核心（Shared Kernel）與通用工具：`modularization/shared-kernel.md`

### 依賴規則與邊界
- 依賴方向與穩定性：`constraints/dependency-direction-and-stability.md`
- 邊界、可見性與封裝：`constraints/boundaries-and-encapsulation.md`
- 介面設計與替換性：`constraints/interface-design-and-substitution.md`
- 模組間通訊（同步/非同步）：`constraints/inter-module-communication.md`

### 測試策略
- 單元測試（Mock Ports/Interfaces）：`testing/unit-testing.md`
- 合約測試（Ports↔Adapters）：`testing/contract-testing.md`
- 整合與端對端：`testing/integration-and-e2e.md`
- 測試資料與固定件（Fixtures）：`testing/test-data-and-fixtures.md`
- 可測性設計準則：`testing/testability-guidelines.md`

### 文件範本與腳手架
- 索引與說明：`templates/README.md`
- 架構說明文件範本：`templates/architecture-doc-template.md`
- 模組 README 範本：`templates/module-readme-template.md`
- ADR 模板：`templates/adr-template.md`
- 檢核表模板：`templates/checklist-template.md`

### ADR（Architecture Decision Records）
- 索引與命名規範：`adr/README.md`
- 範例：`adr/0001-example-decision.md`

### 其他通用文件
- 名詞表與通用語彙：`glossary.md`
- 常見問答（FAQ）：`faq.md`
- 撰寫與貢獻指南：`CONTRIBUTING.md`
- 編碼與格式規範（UTF-8/LF）：`STYLE.md`

---

## 命名與格式規範（摘要）
- 編碼：UTF-8（無 BOM）；行尾：LF（\n）。
- 語言：對使用者溝通一律使用繁體中文；技術名詞與程式碼可保留英文。
- 檔名：使用 kebab-case，英文與縮寫需一致（例如 `anti-patterns.md`）。
- 連結：一律使用相對路徑，避免硬編絕對路徑。

## 路徑慣例（摘要）
- 架構主題置於：`docs/architecture/<topic>/...`
- 橫切主題（測試、依賴、模板）置於對應資料夾：`testing/`、`constraints/`、`templates/`。
- DDD 可依有界上下文延伸子目錄：例如 `ddd/examples/<context-name>/...`。

---

## 導覽建議
- 初次導入：先讀「模組化與分層原則」→ 選擇架構（Clean/Hex/Onion/DDD）→ 套用對應「目錄模板」。
- 進階主題：依需要閱讀「依賴規則與邊界」「測試策略」「ADR」。
- 問題排除：參考各架構的「反模式」與「檢核表」。

---

> 版權與授權：遵循專案根目錄之授權規範（如有）。

