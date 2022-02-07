# Buildpack Tips and Tricks


<!--more-->

2018 年去上海參加 Kubecon 就有聽到過 CNCF 下的 [Cloud Native Buildpacks](https://buildpacks.io/) 的專案。回來也有試玩了一下，概念很不錯，那時候可能整個生態系統比較不建全，試玩了一下就放到旁邊去了

不過自從 [Google Cloud Next 20' - Hands-on Keynote: Building Trust for Speedy Innovation](https://cloud.withgoogle.com/next/sf/onair?session=SVR227#application-modernization) 中也有場次提到 Google 內部已經使用 Buildpack 的服務，之後就有比較用力的試了一下，也導入到自己的工程流程中

> - Google Cloud maintains a set of open source buildpacks for building containers to run > on **GKE**，**Anthos**，& **Cloud Run**
> - **Cloud Build** native support for buildpacks
> - **Cloud Run** direct source deployments w/ buildpacks
> - **Cloud Shell** has pack pre-installed. App Engine builds via buildpacks
> - **Cloud Functions** builds via buildpacks
> - **Cloud Code** deploy to Cloud Run with Buildpacks
> - **Skaffold** native support for buildpacks

在跟同事分享 buildpack 的時候，同事覺得太複雜了。同樣的動作 `Dockerfile` 操作比 buildpack 來的容易簡單，那為什麼還需要導入 buildpack 這個東西把事情弄的複雜? 

ps. (先補充一個血淚: 只拿到前人留下來的 container image，沒有任何 `Dockerfile`, 後序再次開發只有一個慘字，如果一開始使用 buildpack 就不會那麼辛苦)

### Cloud Native Buildpacks

> **Cloud Native Buildpacks**
transform your application source code into images that can run on any cloud.

先從 buildpack 的目的來說，它其實是定義了一套如何從程式源始碼建立 container image 的 [buildpacks/spec: Specification for Cloud Native Buildpacks](https://github.com/buildpacks/spec)

> This specification defines interactions between a platform, a lifecycle, a number of buildpacks, and an application
> 1. For the purpose of transforming that application into an OCI image and
> 2. For the purpose of developing or executing automated tests on that application.
> 
> A **buildpack** is software that partially or completely transforms application source code into runnable artifacts.
> 
> A **lifecycle** is software that orchestrates buildpacks and transforms the resulting artifacts into an OCI image.
> 
> A **platform** is software that orchestrates a lifecycle to make buildpack functionality available to end-users such as application developers.

總的來說，buildpack 的目的是建立一個通用的規範， builder (Google, Heroku, Paketo) 除了實作這個規範之外也會增加一些功能來支援自己的產品(如 Google 的 gcr.io/buildpacks/builder:v1 builder 就有支援 Google Cloud function)。一般的情況就是選擇一個自己適合的 builder 直接使用

```sh
# Examples:
pack build test_img --path apps/test-app --builder cnbs/sample-builder:bionic
```

當有特別的需求是可以自定義 buildpack 來完成，但開發者也必需依照同樣的規範的來實作自己的 buildpack；也因為 buildpack 需要按照規範來寫，不像 `Dockerfile` 這樣可以隨便寫(誤)，入門會有一點門檻，但是效益上會體現在如 container image 的安全性，大量產出 container image 的 CI/CD 流程上

聽起來好像很方便又覺得那邊卡卡的，我來說說一些心得

**Pros**
- 不用寫 `Dcokerfile`
- 一般的情況可以無惱的使用，可以把精力放在商業邏輯上
- container image 的安性性由 builder 幫你處理
- CI/CD 流程的整合: skaffold, cloud build, tekton 都有支援

**Cons**
- 因為要尊循規範, 自定義的入門門檻還是有一些
- 無惱 build container image 看似很美好，遇到 builder (Google, Heroku, Paketo) 不提供的部份就會很難受(踩到的坑後面再說)
- 如果想 build 出 container image size 小的部份得看 builder 有沒有支援，(如: paketobuildpacks/builder:tiny)

### 坑

現在來談談最近遇到的坑，先來看看現在 pack 建議的 builders，有 Google、Heroku 和 Paketo 三個，使用者可以跟據自己的需求選擇適合的 builder

```sh
 pack builder suggest
Suggested builders:
	Google:                gcr.io/buildpacks/builder:v1      Ubuntu 18 base image with buildpacks for .NET, Go, Java, Node.js, and Python
	Heroku:                heroku/buildpacks:18              Base builder for Heroku-18 stack, based on ubuntu:18.04 base image
	Heroku:                heroku/buildpacks:20              Base builder for Heroku-20 stack, based on ubuntu:20.04 base image
	Paketo Buildpacks:     paketobuildpacks/builder:base     Ubuntu bionic base image with buildpacks for Java, .NET Core, NodeJS, Go, Ruby, NGINX and Procfile
	Paketo Buildpacks:     paketobuildpacks/builder:full     Ubuntu bionic base image with buildpacks for Java, .NET Core, NodeJS, Go, PHP, Ruby, Apache HTTPD, NGINX and Procfile
	Paketo Buildpacks:     paketobuildpacks/builder:tiny     Tiny base image (bionic build image, distroless-like run image) with buildpacks for Java Native Image and Go
```

#### 坑1 paketobuildpacks/builder:tiny

> 目前不支持 golang module 中使用 private repo

```sh
fatal: could not read Username for 'https://github.com': terminal prompts disabled
      Confirm the import path was entered correctly.
      If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
```

依據 [Private Repository Support · Issue #3 · paketo-buildpacks/go-mod-vendor](https://github.com/paketo-buildpacks/go-mod-vendor/issues/3) 中討論的部份，如果是使用 Golang 語言且 Go Module 中有使用到 private repo 就會遇到權限不足的問題，issues 討論中也表示目前並不支持 private repo，不過有提出 `pack` 可以 inject SSH key (不過我自己是沒有試成功)

這個只適合想用 buildpack 產出 tiny 大小的 container image (Golang)，不過前提是沒有使用 private repo 

#### 坑2 gcr.io/buildpacks/builder:v1

> gcr.io/buildpacks/gcp/run:v1 不包含 `/usr/share/zoneinfo`，如果 Golang 中有使用到 `loc, _ := time.LoadLocation("Asia/Taipei")` 會報錯，不過 paketobuildpacks/builder:tiny 的 run image 有阿 !!!!

在我們用 gcr.io/buildpacks/builder:v1 產出 container image 後執行後發現報錯，root cause 是 container image 並不包含 `/usr/share/zoneinfo` 目錄導致報錯, `pack inspect <container-image>` 後得知 run image 是 gcr.io/buildpacks/gcp/run:v1，而 gcr.io/buildpacks/gcp/run:v1 本身就沒有包含 `/usr/share/zoneinfo` 目錄，這時候就必需擴充 gcr.io/buildpacks/gcp/run:v1 中安裝 `tzdata`

```dockerfile
FROM gcr.io/buildpacks/gcp/run:v1
USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
  tzdata && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*
USER cnb
```

產出擴充的 run image

```sh
docker build -t gcr.io/retailbase-dev/ext-run:v1 -f run.Dockerfile .
```

然後在 pack 中代入 擴充後有 run time

```sh
pack build gcr.io/retailbase-dev/retailbase-organizationsvc:$SHORT_SHA --builder gcr.io/buildpacks/builder:v1 --env GOOGLE_BUILDABLE=cmd/organizationsvc/main.go --run-image gcr.io/retailbase-dev/ext-run:v1
```

#### 2021/04/08 update 
在發 issue [Include tzdata in base images · GoogleCloudPlatform/buildpacks@50539bf](https://github.com/GoogleCloudPlatform/buildpacks/commit/50539bf027fcb907e72f65e50e3ce3904b8821fb) 之後，Google 已經在 base image 中加上了 `tzdata`，這樣就不需要擴充 runtime 來解 Golang panic 的 bug，直接使用原來的 `gcr.io/buildpacks/builder:v1` 即可

#### 坑三 gcr.io/cloud-builders/gcloud

> cloudbuild gcr.io/cloud-builders/gcloud 中有整合 pack 的參數，不過沒有辦法追加 runtime 的設定，就算我們擴充了 run time 也無法代入

```sh
gcloud builds submit [[SOURCE] --no-source] [--async] [--no-cache]
        [--disk-size=DISK_SIZE] [--gcs-log-dir=GCS_LOG_DIR]
        [--gcs-source-staging-dir=GCS_SOURCE_STAGING_DIR]
        [--ignore-file=IGNORE_FILE] [--machine-type=MACHINE_TYPE]
        [--region=REGION] [--substitutions=[KEY=VALUE,...]] [--timeout=TIMEOUT]
        [--worker-pool=WORKER_POOL]
        [--config=CONFIG; default="cloudbuild.yaml"
          | --pack=[builder=BUILDER],[env=ENV],[image=IMAGE]
          | --tag=TAG, -t TAG] [GCLOUD_WIDE_FLAG ...]
```

gcr.io/cloud-builders/gcloud 使用 pack 的相關參數只有 builder, env, 及 image

```yaml
 - id: pack build export
    name: gcr.io/cloud-builders/gcloud
    args:
      - builds
      - submit
      - --pack=builder=gcr.io/buildpacks/builder:v1,env=GOOGLE_BUILDABLE=cmd/export/main.go,image=gcr.io/retailbase-dev/retailbase-export:$SHORT_SHA
```

所以就必需透過 [GoogleCloudPlatform/cloud-builders-community: Community-contributed images for Google Cloud Build](https://github.com/GoogleCloudPlatform/cloud-builders-community) 中的方式，自行打包 pack 來使用

```yaml
  - name: gcr.io/retailbase-dev/pack:v0.18.0
    args:
      - build
      - gcr.io/retailbase-dev/retailbase-export:$SHORT_SHA
      - --builder=gcr.io/buildpacks/builder:v1
      - --env=GOOGLE_BUILDABLE=cmd/export/main.go
      - --run-image=gcr.io/retailbase-dev/ext-run:v1
```

如何自定義 buildpack 就寫在另一篇文章
#### 2021/04/09 update

跟據 [GoogleCloudPlatform/buildpacks](https://github.com/GoogleCloudPlatform/buildpacks#using-with-google-cloud-build) 的說明，skaffold team 有提供 `gcr.io/k8s-skaffold/pack` pack 的 instance 可以直接使用，也就是說不需要自行編譯 community buildpack pack 的方式

```yaml
steps:
- name: gcr.io/k8s-skaffold/pack
  entrypoint: pack
  args:
    - build
    - --builder=gcr.io/buildpacks/builder
    - --publish=gcr.io/$PROJECT_ID/sample-go:$COMMIT_SHA
```

### Reference
- [Cloud Native Buildpacks](https://buildpacks.io/): Buildpack 官方網站
- [GoogleCloudPlatform/buildpacks: Builders and buildpacks designed to run on Google Cloud's container platforms](https://github.com/GoogleCloudPlatform/buildpacks): Google 版的 buildpack
- [Paketo Buildpacks - Paketo Buildpacks](https://paketo.io/)
- [Buildpacks | Heroku Dev Center](https://devcenter.heroku.com/articles/buildpacks): Heroku 版的 Buildpack
- [gcloud builds submit  |  Cloud SDK Documentation  |  Google Cloud](https://cloud.google.com/sdk/gcloud/reference/builds/submit)
