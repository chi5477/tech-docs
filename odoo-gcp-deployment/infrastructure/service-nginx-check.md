# Compute Engine Nginx 驗證流程

## 安裝與啟動 Nginx
1. `sudo apt update`
2. `sudo apt install nginx -y`
3. `sudo systemctl enable nginx`
4. `sudo systemctl start nginx`

## 服務狀態檢查
- 透過 `systemctl status nginx` 確認服務是否為 active (running)。
- 若需要常駐檢視，可搭配 `sudo journalctl -u nginx -f` 追蹤最新日誌。

## 基礎可用性測試
- 在 VM 內部執行 `curl http://127.0.0.1` 應回傳預設的 Nginx welcome 頁面 HTML。
- 若對外驗證，需要確認防火牆規則或負載平衡器已開放 HTTP/80 流量，再由本機瀏覽器訪問 `http://<VM-內部或負載平衡器 IP>`。

## 後續處理
- 測試完成後若無需 Nginx，可使用 `sudo systemctl stop nginx` 停止服務，或 `sudo apt remove nginx -y` 移除。