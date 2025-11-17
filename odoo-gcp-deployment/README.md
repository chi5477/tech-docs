# Odoo GCP 部署文件集

## 目的
彙整在 GCP 建立 Odoo 18.0 測試／開發環境的決策、設定與驗證紀錄。內容涵蓋 Compute Engine 基礎配置、平台調整、資源規模估算，以及相關截圖備查。

## 目錄結構與重點文件
- `infrastructure/`：Compute Engine 層的設定與驗證。
  - `gcp-instance-baseline.md`：磁碟、快照、安全性、進階功能的初始配置。
  - `gcp-network-interface.md`：VPC、子網路、IP 型態與服務等級的說明。
  - `compute-access.md`：透過 gcloud / SSH / IAP 連線 VM 的做法，含 VS Code Remote 代理設定。
  - `service-nginx-check.md`：以 Nginx 驗證 VM 對外服務可達性的操作手冊。
  - `same-terraform.tf`：對應 Terraform 範例（僅允許更名或引用，避免直接修改）。
- `platform/`：作業系統、容器與 Odoo 平台層設定。
  - `linux-distro-selection.md`、`ubuntu.md`、`ubuntu-docker-install.md`。
  - `odoo-websocket-gevent.md`：整理 Gevent / Werkzeug 雙埠行為與 `/websocket` 代理需求。
  - `docker-bind-volume-diff.md`：比較 Windows 與 Linux bind volume 權限差異與處理方式。
- `sizing/`
  - `odoo-machine-sizing.md`：依 Cloudpepper 經驗法則估算 worker、CPU、RAM 與機型選擇。
- `images/`：依主題分類的截圖資源（`compute/`、`network/`、`storage/`、`security/`、`advanced/`、`supplemental/`）。

## 圖片對應
- `images/storage/disk-settings.png`、`images/storage/disk-type-comparison.png`：對應 `infrastructure/gcp-instance-baseline.md` 的磁碟選擇說明。
- `images/storage/snapshot-schedule.png`：預設快照排程畫面，對應備份策略。
- `images/network/network-interface-main.png`、`images/network/network-interface-edit.png`：對應 `infrastructure/gcp-network-interface.md` 的主要／進階設定。
- `images/security/security-settings.png`、`images/advanced/advanced-settings.png`：對應安全性與進階設定。
- `images/compute/machine-series-selection.png`、`images/compute/machine-type-e2-small.png`：對應 `sizing/odoo-machine-sizing.md` 的機型評估。
- `images/supplemental/snapshot-schedule-alt.png`：快照排程備用截圖，暫未直接引用，可於後續報告或教學使用。

## 使用建議
1. 依序閱讀 `infrastructure/` → `platform/` → `sizing/`，確保硬體、網路與平台設定一致。
2. 若需要 IaC，自 `infrastructure/same-terraform.tf` 複製範例並調整專案參數後使用。
3. 新增截圖時放入對應子資料夾，並在本 README 的「圖片對應」區更新註記。
4. 若發現內容需要驗證或補充（例如 SSH / WebSocket 原理、VS Code Remote 測試結果），請在原文件的 TODO 區段補齊。

