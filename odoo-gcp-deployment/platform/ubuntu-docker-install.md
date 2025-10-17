# Ubuntu 安裝 Docker 指南

- 參考來源：[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

## 1. 移除舊版套件（如已安裝）
    sudo apt-get remove docker docker-engine docker.io containerd runc

## 2. 建立 apt 套件庫
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg lsb-release
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    source /etc/os-release
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu ${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update

## 3. 安裝 Docker Engine 套件
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

## 4. 驗證安裝
    sudo docker run hello-world
成功看到歡迎訊息代表 Docker Engine 已可使用。

## 5. （選用）非 root 使用者操作 Docker
    sudo usermod -aG docker $USER
    newgrp docker
執行後使用者即可直接執行 docker 指令。
