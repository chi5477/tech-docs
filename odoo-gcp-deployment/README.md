# Odoo GCP 部署文件集

## 目的
彙整在 GCP 建立 Odoo 18.0 測試／開發環境的所有決策、設定與參考資料，作為之後擴充或轉為正式環境的基礎文件。

## 目錄結構
- infrastructure/：Compute Engine 相關基礎設定與 Terraform 範例。
  - gcp-instance-baseline.md：磁碟、快照、安全性、進階設定選擇。
  - gcp-network-interface.md：網路介面與進階欄位說明。
  - compute-access.md：連線步驟與首次 SSH 金鑰流程。
  - same-terraform.tf：對應的 Terraform 範例（僅允許更名或引用）。
- platform/：作業系統選型與 Ubuntu 相關操作。
  - linux-distro-selection.md、ubuntu.md、ubuntu-docker-install.md。
- sizing/：Odoo worker、CPU、記憶體與機型選擇原則。
- images/：依主題分類的截圖資源。
  - compute/、network/、storage/、security/、advanced/、supplemental/。

## 圖片對應
- images/storage/disk-settings.png、disk-type-comparison.png：對應 infrastructure/gcp-instance-baseline.md 的磁碟選型說明。
- images/storage/snapshot-schedule.png：預設快照排程截圖，對應備份策略。
- images/network/network-interface-main.png、network-interface-edit.png：對應 infrastructure/gcp-network-interface.md。
- images/security/security-settings.png 與 images/advanced/advanced-settings.png：安全性與進階設定面板。
- images/compute/machine-series-selection.png、machine-type-e2-small.png：對應 sizing/odoo-machine-sizing.md 的機型評估。

## 補充圖庫
- images/supplemental/snapshot-schedule-alt.png：快照排程的備用截圖，暫未直接引用，可於後續報告或教學中視需要使用。

## 使用建議
1. 依據文件順序確認硬體規格、磁碟／安全設定，再進行 OS 與 Docker 配置。
2. 若需要 IaC，自 infrastructure/same-terraform.tf 複製範例並調整專案參數後使用。
3. 有新增截圖時請放入對應主題資料夾，並在相關文件或本檔案的圖片對應區更新註記。
