# Odoo WebSocket 與 Gevent 行為整理

> 注意：以下內容依據目前經驗彙整，尚未逐點實驗驗證，但合理性高。

## Server 啟動模式
- `gevent_port` 啟用後，Odoo 會同時啟動兩個 HTTP server：Werkzeug（傳統 HTTP）與 Gevent（事件驅動）。
- Gevent Server 額外支援 WebSocket、`/longpolling` 與 bus 相關的即時通訊功能。

## WebSocket 控制器
- `/websocket` 的 controller 程式碼是共用的，Gevent 與 Werkzeug 皆會載入。
- 只有 Gevent server 的 `environ['socket']` 會存在；Werkzeug 並不提供此值。

## 常見錯誤
- 若使用錯誤的 port（例如連到 Werkzeug 預設 8069）呼叫 `/websocket`，會出現：
  ```text
  RuntimeError: Couldn't bind the websocket. Is the connection opened on the evented port (8072)?
  ```
- 這表示請求未經由 Gevent port（預設 8072）處理。

## Nginx 轉發建議
- Nginx（或其他前端代理）必須將所有 `/websocket`、`/longpolling`、bus 相關流量自動轉送到 Odoo 的 `gevent_port`（預設 8072）。
- 其餘 HTTP 流量可照常轉送到 Werkzeug 服務的 port（預設 8069）。

## 待確認事項
- 雙 server 模式下不同 port 的健康檢查策略。
- 是否需要針對 WebSocket 連線調整代理的 keep-alive 或超時參數。