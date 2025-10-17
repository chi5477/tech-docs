# GCP Artifact Registry：映像建置與推送指引

本文件整理如何將映像推送至 GCP Artifact Registry，並確保版本一致且路徑正確，可供 Cloud Run/Terraform 參考使用。

## 0) 共用變數（單一真實來源）

```bash
# 請依實際環境替換
PROJECT_ID="<your-project>"
REGION="asia-east1"
REPO="<your-repo>"       # 例如 erp
IMAGE="<your-image>"     # 例如 odoo 或 nginx
TAG="$(date +%Y%m%d-%H%M%S)"  # 或用 git SHA: $(git rev-parse --short HEAD)

IMAGE_URI="$REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:$TAG"
```

之後所有 build/push/deploy 僅使用 `$IMAGE_URI`，避免同時產生多個不同路徑或 tag 的映像。

## 1) 準備 Artifact Registry

```bash
# 啟用 API（一次性）
gcloud services enable artifactregistry.googleapis.com

# 建立 Docker repository（如已存在可略）
gcloud artifacts repositories create "$REPO" \
  --repository-format=docker \
  --location="$REGION"
```

重要：請先以正確的 Google 帳戶登入（需具有該專案的 Artifact Registry 推送權限），否則 push 會被拒絕。

```bash
# 登入正確 Google 帳戶（會開瀏覽器驗證）
gcloud auth login

# 設定專案（確保後續操作都落在正確的 PROJECT_ID）
gcloud config set project "$PROJECT_ID"

# 重新設定 Docker 認證給對應區域的 Artifact Registry
gcloud auth configure-docker "$REGION-docker.pkg.dev"
```

## 2) 建置與推送（二擇一）

方式 A：本機 Docker build & push

```bash
docker build -t "$IMAGE_URI" .
docker push "$IMAGE_URI"
```

方式 B：Cloud Build 代為建置 & 推送（免本機 Docker 登入）

```bash
gcloud services enable cloudbuild.googleapis.com
gcloud builds submit --tag "$IMAGE_URI" .
```

無論 A 或 B，僅會產生同一個 `$IMAGE_URI` 版本的映像。

## 3) 驗證映像與標籤

```bash
gcloud artifacts docker images list \
  "$REGION-docker.pkg.dev/$PROJECT_ID/$REPO" \
  --include-tags
```

## 4) 清除錯誤映像/tag（如不小心多推）

```bash
# 注意：會刪除該 tag 對應的映像，可加 --quiet 靜默
gcloud artifacts docker images delete \
  "$REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:<WRONG_TAG>" \
  --delete-tags
```

## 常見陷阱與說明

- build 與 push 指向不同的專案/倉庫/路徑，會導致「看起來變兩份」。
  - 例如 build 標記到 `asia-east1-docker.pkg.dev/<A>/cloud-run-source-deploy/erp`，
    但 push 卻推到 `asia-east1-docker.pkg.dev/<B>/cloud-qwer-source-deploy/erp`。
  - 解法：統一以 `$IMAGE_URI` 作為唯一來源；建置、推送、部署皆使用同一路徑與 tag。

## 與本專案整合建議

- 建議建立兩個映像：
  - Odoo：`$REGION-docker.pkg.dev/$PROJECT_ID/$REPO/odoo:$TAG`
  - Nginx：`$REGION-docker.pkg.dev/$PROJECT_ID/$REPO/nginx:$TAG`
- Terraform 變數：將上列完整 URL 帶入 `infra/variables.tf` 所需的 `odoo_image`、`nginx_image`。
