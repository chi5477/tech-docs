# Ubuntu 24.04 LTS Minimal 環境設計

## 選型理由
- 採用 **Ubuntu 24.04 LTS Minimal**，兼具 LTS 長期維護穩定與 Minimal 的輕量基底，適合雲端長期部署。
- 無需 GUI，透過 VS Code Remote（SSH / Dev Containers）即可在同一台 VM 上進行 Odoo 18.0 的部署與開發測試。
- Minimal 版本維持套件精簡，降低資源佔用，後續按需安裝 Docker、開發工具即可支援日常工作流程。

## Odoo 與 Python 多工策略
- Odoo 執行核心以 Python 為主，受制於 **Global Interpreter Lock (GIL)**：同一個 Python 行程內的多執行緒無法真正同步在多核心上並行，僅能快速切換達成類似並行的效果。
- Odoo 透過 **多 worker 行程**（multi-process）繞開 GIL 限制，使每個 worker 可以對應到獨立 CPU 核心，實際獲得多核心效益。
- 需要高併發或背景排程時，可依照並行需求調整 workers、cron、gevent 等數量，以平衡資源使用與吞吐量。

## 執行緒與核心的思考
- CPU 核心：實體的運算單元，好比高速公路的車道數，決定可真正同時執行的工作數量。
- 執行緒（Thread）：作業系統層級的邏輯工作單位，如同車輛在車道間切換；高負載任務仍需佔用實體車道，無法無限制擴充。
- Python 的 GIL 讓單一行程內的執行緒無法同時占用多個核心，因此需以多個行程（Odoo workers）對應多核心，才能充分運用硬體資源。


## 環境拓樸概覽
```
VS Code (client)
  |- Remote-SSH / Dev Containers
  |    |- Ubuntu 24.04 LTS Minimal (server, no GUI)
  |         |- Docker (runtime)
  |         |    |- odoo:18 (workers > 0)
  |         |    |    |- HTTP workers x N
  |         |    |    |- LiveChat (gevent / websocket)
  |         |    |    |- Cron workers
  |         |    |- postgres:15/16
  |         |    |- nginx:stable (reverse proxy)
  |         |- Project devcontainers (per project)
  |- VS Code extensions and tooling
```
