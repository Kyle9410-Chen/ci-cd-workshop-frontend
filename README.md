# ci-cd-workshop-frontend

Todo List 前端，使用 Vite + React + TypeScript，以 TanStack Query 管理 server state。

## 本機開發

需求：Node.js 24（版本記錄在 `.nvmrc`，可用 `nvm use` 切換）。需要先啟動後端（見 `ci-cd-workshop-backend`）。

```bash
npm install
npm run dev
```

開啟 http://localhost:5173

## 測試與靜態檢查

```bash
npm run lint
npm run typecheck
npm run format:check
npm test -- --run
npx vitest --run src/features/todos/filterState.test.ts
npm run build
```

## Docker

```bash
cp .env.example .env
docker compose up --build -d
```

開啟 http://localhost:8080

容器內的 Nginx 只負責提供靜態檔案。API 位址在 **build time** 由 `VITE_API_BASE_URL` 寫入 bundle，修改後必須重新 build image。

## CI/CD

`.github/workflows/main.yml` 在每次 push（任何 branch）時執行 Lint 與 Test；只有 `main` 在兩者都通過後，才會 build image 並推到 Docker Hub，tag 為 `latest` 與 commit SHA。

需要在 GitHub repo 的 **Settings → Secrets and variables → Actions** 設定：

| 類型     | 名稱                 | 內容                                                     |
| -------- | -------------------- | -------------------------------------------------------- |
| Variable | `DOCKERHUB_USERNAME` | Docker Hub 帳號名稱                                      |
| Secret   | `DOCKERHUB_TOKEN`    | Docker Hub 的 Personal Access Token（權限 Read & Write） |

Image：`<DOCKERHUB_USERNAME>/ci-cd-workshop-frontend`

CD 產生的 image 以 `VITE_API_BASE_URL=/api` build，前端會呼叫同網域的 `/api`，部署時需要由 reverse proxy 將 `/api` 轉給 backend。

## 環境變數

| 名稱                | 預設值                      | 說明                       |
| ------------------- | --------------------------- | -------------------------- |
| `VITE_API_BASE_URL` | `http://localhost:8000/api` | 後端 API 位址（build arg） |
| `WEB_PORT`          | `8080`                      | docker compose 對外的 port |
