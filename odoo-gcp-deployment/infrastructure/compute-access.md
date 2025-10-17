# GCP Compute Engine 連線方式

## 1. 使用 gcloud CLI 建立 SSH 連線
- 指令格式：`gcloud compute ssh --zone "<zone>" "<instance-name>" --project "<project-id>"`
- 首次連線若本機尚無 SSH 金鑰，會出現以下警示：
  - WARNING: The private SSH key file for gcloud does not exist.
  - WARNING: The public SSH key file for gcloud does not exist.
  - WARNING: The PuTTY PPK SSH key file for gcloud does not exist。
  - WARNING: You do not have an SSH key for gcloud.
  - WARNING: SSH keygen will be executed to generate a key。
- gcloud 會自動執行 `ssh-keygen` 生成一組新的金鑰，並將公鑰上傳到目標 Compute Engine，之後即可透過命令列直接連線。

## 2. Windows 端使用 Plink 連線
- 金鑰產生後，gcloud 會提示 Plink（PuTTY）是否需要記住此主機的指紋。
- 在 Windows 系統執行 `plink` 連線時，首次會出現「是否信任並儲存主機鍵」的詢問，選擇「是」即可在後續免除提示。

## 3. 傳統 SSH 指令透過 IAP
- 若 Compute Engine 無固定對外 IP，可利用 IAP 建立管道：
  1. 執行 `gcloud compute config-ssh` 產生基本 SSH config。
  2. 在 `~/.ssh/config` 追加設定：
```
Host <HOST NAME>
    ProxyCommand gcloud.cmd compute ssh --zone <ZONE> --project <PROJECT> --tunnel-through-iap -- -W %h:%p
```
  3. 之後即可使用 `ssh <HOST NAME>`，透過 IAP 管道連線到 Compute Engine。

## 4. VS Code Remote IAP 設定
- 為避免 `gcloud compute ssh` 自動開啟遠端終端機並確保雙向通道，改採 `start-iap-tunnel`：
```
Host <HOST NAME>
    HostName <VM-NAME>
    User <USER>
    IdentityFile ~/.ssh/google_compute_engine
    ProxyCommand gcloud.cmd compute start-iap-tunnel %h %p --project=<PROJECT> --zone=<ZONE> --listen-on-stdin
```
- 已測試可透過 VS Code Remote 正常連線。

## 5. SSH 技術原理（待補）
- TODO：補充 SSH 公私鑰交換流程、金鑰註冊與主機指紋驗證原理。