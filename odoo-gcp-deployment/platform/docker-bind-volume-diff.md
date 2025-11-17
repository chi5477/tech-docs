# Docker Compose Bind Volume 差異（Windows vs Linux）

> 根據目前驗證所得，描述 Docker Compose 在兩種平台下使用 bind volumes 的主要差異與處置方式。

## 行為差異
- **Linux（原生 Docker）**  
  容器直接使用宿主機的 filesystem namespace，檔案會保留真實的 UID/GID 與權限，可透過 `chown`、`chmod` 進行權限管理。
- **Windows（Docker Desktop）**  
  Volume 透過虛擬層（WSL2/Hyper-V）共享，宿主檔案缺乏 UID/GID 概念，容器內看到的檔案通常為 `root:root`，權限調整不會永久生效。

## 建議處置
- 在 Linux 環境下，先於宿主機授與正確的擁有者與權限，再重新 build/啟動服務，容器即可沿用調整後的設定。
- 在 Windows 環境若需一致行為，建議避免依賴宿主檔案權限，可改用 named volumes 或在容器啟動腳本中以 `chmod` 開放存取。

