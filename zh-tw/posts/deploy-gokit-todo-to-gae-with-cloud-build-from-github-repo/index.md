# Deploy Gokit Todo to Gae With Cloud Build From Github Repo


<!--more-->

## gokit-todo

https://github.com/cage1016/gokit-todo

>todomvc full stack demo project. react + backend API by gokit microservice toolkit. include unit test, integration test, e2e test, github action ci

https://github.com/cage1016/gokit-todo-frontend

> gokit todo frotnend: React todomvc

{{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/demo.gif" alt="gokit-todo demo">}}

`gokit-todo` (使用 Postgres 為資料庫的後端微服務) 和 `gokit-todo-frontend` (以 React todomvc 的前端) 在部署在 Kubernetes + Ingress (Istio / Nginx-ingress) 的應用程式。或是可以用 docker-compose 的方式來執行。有興趣的朋友可以到 Github Repo 查看操作方式

## 移轉 gokit-todo gokit-todo-frontent 至 Google App Engine 及 Cloud SQL

在我們將 gokit-todo 從 kubernetes 遷移到 Google App Engine 上前，我們需要先對 Google App Engine 有一些了解。來確保我們這一個想法是可行的

1. Google App Engine 是一個 PasS (platform as a service) 層級的服務，也就是我們只需要專注在應用程式的開發。所以我們可以重複使用 gokit-todo, gokit-todo-frontend 中的程式碼透過 `app.yaml` 配置部署到 Google App Engine ✅
1. Google App Engine 支援的 standard-runtime (`Python`, `Java`, Node.`js`, `PHP`, `Ruby`, `Go`) 及 flexible-runtime (`Go`, `Java 8`, `PHP 5/7`, `Python 2.7/3.6`, `.NET`, `Node.js`, `Ruby`, `Custom runtime`)。
   - gokit-todo 作為單純 API 後端只需要使用 standard-runtime Go 1.16 即可 ✅
   - gokit-todo-frontentd 使用 React 編寫，我們也可以選擇使用 standard-runtime Node.js 16 即可 ✅
1. Gokit-todo 微服務介接的是 Postgres，在 Google App Engine 的環境中可以使用 Cloud SQL 來替代資料庫的腳色。要注意的地方是，得使用 Cloud SQL proxy 提供的 driver `cloudsqlpostgres`，這一個 driver 會幫你處理一些接介 Cloud SQL 上的認證問題，所以資料庫的部份也沒問題 ✅
1. Google App Engine 支援多個獨立的服務，再透過 `disptach.yaml` 的設定來串起多個服務之間的關係。Gokit-todo backend、Gokit-doto frontend 在 `dispatch.yaml` 的設定之下就可以達到微服務的效果 ✅
1. CI/CD 的部份可以使用 Google Cloud Build 來編寫，也可以整合 Github。要注意的部份就是 Cloud Build 設定服務帳號 `project-number@cloudbuild.gserviceaccount.com` 需要給定足夠的權限 ✅
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/cloudbuild permission.jpg" alt="Google Cloud Build permission">}}

**Architecture**

{{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/placeholder.png" alt="gokit-todo-gae architecture">}}

上圖為基本架構圖，我們新建立一個專案 `gokit-todo-gae` 並將應用程式 `gokit-todo` 及 `gokit-todo-frontend` 加到 git submodule. 在專案中使用 git submodule 可以有效避免干擾原來的應用程式原始碼

## Google App Engine

Google App Engine 的部份在這一次的情境中相對的很單純，基本上就是啟用，後序的部署我們都是使用 Cloud Buiild 來串接

```bash
gcloud app create --region=asia-east1
```

## Cloud SQL

我們可以透過 Google Cloud Console 或是 `gcloud sql instances create` command 來建立 Cloud SQL 的實例。這邊我們使用 `gcloud` command

```bash
gcloud sql instances create todo --database-version=POSTGRES_11 --cpu=1 --memory=3840MiB --region=asia-east1 --root-password=password --storage-size=10GB --storage-type=SSD
```

Cloud Sql 實例建立好之後，我們需要建立一個資料庫

```bash
gcloud sql databases create todo -i todo
```

資料庫部份的設置基本上就完成了。Cloud SQL 實例的 IP 預設為公開，本機開發的時候就可以使用 Cloud SQL Proxy 連接方便使用。當然可以啟用私人 IP 搭配 VPC Network 來使用。另外我們資料庫的連線方式為 `<project-id>:<region>:<Database-name>`，這個後序會使用到

## Google Cloud Build

由於這一個專案 CI/CD 都是由 Google Cloud Build 串接起來的，所以這一塊比較複雜一點

1. 二個 git submodule: 後端 gokit-todo (api)，前端 gokit-todo-frontend (default)

      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/submodule trigger.jpg" alt="submodule triggers">}}

      > 在 Cloud Build 步驟中使用 RESTFul API 來驅動另一個 Cloud Build trigger 是一個很有用的技巧

      ```bash
      curl -d '{"branchName":"master"}' -X POST -H "Content-type: application/json" \
          -H "Authorization: Bearer $(gcloud config config-helper --format='value(credential.access_token)')" \
          https://cloudbuild.googleapis.com/v1/projects/<gcp-project>/triggers/<cloudbuild-trigger-id>:run
      ```

1. 透過 `cloudbuild.api.yaml` 及 `cloudbuild.default.yaml` 來觸發觸發條件 `gokit-todo` 和 `gokit-todo-frontend`

      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/api default trigger.jpg" alt="api/default triggers">}}

1. Google App Engine dispatch 的路由設定 `dispatch.yaml` 則透過 `cloudbuild.dispatch.yaml` 來部署

__全部 Cloud Build 的設定__
{{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/cloudbuild-trigger.jpg" alt="Cloud Build triggers">}}

## gokit-todo-gae

**[cage1016/gokit-todo-gae](https://github.com/cage1016/gokit-todo-gae)**

```bash
.
├── api                       // gokit-todo submodule as api service
├── default                   // gokit-todo-frontend as default service
├── .gitmodules
├── cloudbuild.api.yaml       // deploy api service (Manual)
├── cloudbuild.default.yaml   // deploy default service (Manual)
├── cloudbuild.dispatch.yaml  // deploy dispatch.yaml
└── dispatch.yaml             // gokit-todo-gae dispatch yaml
```

我們可以看到上述專案的檔案架構跟之前提到的架構圖是一致的，操作流程如下

1. 前端人員將程式推送至 Github 倉庫 `gokit-todo-frontend` 👉 觸發條件 `gokit-todo-frontend` 會被觸發進行對應的任務, 並在最後一個步驟透過 RESTFul 觸發觸發條件 `gokit-todo-gae-deploy-default` 👉 觸發條件 `gokit-todo-gae-deploy-default` 會進行編譯部署應用程式 `gokit-todo-frontend` 到 Google App Engine

      __cloudbuild.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-frontend-cloudbuild.yaml.jpg" alt="gokit-todo-frontend cloudbuild.yaml">}}

      __cloudbuild.default.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-gae-cloudbuild.default.yaml.jpg" alt="gokit-todo-gae deploy default service">}}

1. 後端人員將程式推送至 Github 倉庫 `gokit-todo` 👉 觸發條件 `gokit-todo` 會被觸發進行對應的任務(應用程式測試), 並在最後一個步驟透過 RESTFul 觸發觸發條件 `gokit-todo-gae-deploy-api` 👉 觸發條件 `gokit-todo-gae-deploy-api` 會進行編譯部署應用程式 `gokit-todo` 到 Google App Engine

      __cloudbuild.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-cloudbuild.yaml.jpg" alt="gokit-todo cloudbuild.yaml">}}

      __cloudbuild.api.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-gae-cloudbuild.api.yaml.jpg" alt="gokit-todo-gae deploy api service">}}

1. Google App Engine dispatch routing 設定也是很容易可以更新。只要將修改後的 `dispatch.yaml` 推送至 Github 倉庫 `gokit-todo-gae` 👉 觸發條件 `gokit-todo-gae-deploy-dispatch` 就會將新的 dispatch routing 部署到 Google App Engine

      __cloudbuild.dispatch.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/cloudbuild.dispatch.yaml.jpg" alt="gokit-todo-gae deploy dispatch.yaml">}}

## 心得

Google App Engine 還是很好用的，由其是 standard-runtime 每天有 28 小時實例的免費額適合簡單的專案。再以 `gokit-todo` + `gokit-todo-frontend` 之前部署在 Kubernetes 上的應用來說，都是可以在不修改程式碼的基礎中加上 Google App Engine 需要的設定檔 `app.yaml` 就可以部署，Github 也可以很好的跟 Google Cloud Build 一起協同工作。

**Q**
Github 本身就有自己的 CI/CD 系統 Github Action，為什麼還需要使用 Google Cloud Buil ? </br>
**A**
Github Action 也是可以使用 curl 來驅動父層的 Google Cloud Build 的 tirgger，在 Github action 得自行處理權限問題，都在 Google Cloud Platform 的環境中不用特別處理

{{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-gae.gif" alt="gokit todo GAE">}}

**source code**
- `gokit-todo` https://github.com/cage1016/gokit-todo
- `gokit-todo-frontend` https://github.com/cage1016/gokit-todo-frontend
- `gokit-todo-gae` https://github.com/cage1016/gokit-todo-gae
