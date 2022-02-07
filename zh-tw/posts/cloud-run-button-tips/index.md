# Cloud Run Button Tips


<!--more-->

## Cloud Run

最近為了節省 GCP 服務上的花費，將 GKE 的 cluster 的機器等級調低，相對應的是遷出部份服務到 Google managed 中，也因此重新將目光投射回 `Cloud Run`

Cloud Run 是算是 Goolge Cloud Platfrom 中 serverless (以 Knative 標準打造) 服務的其中一項，開發人員期本上只要掌握打包 Container Image 的技能就可以使用 Cloud Run，如果你不知道 Cloud Run 可以作什麼，下面官方的影片可以看一下

{{< youtube gx8VTa1c8DA >}}

## Container images

剛剛提到開發人員只需要掌握 Container Image 打包的技能就可以入門 Cloud Run。而打包 Container Image 的方式可以從編寫 `Dockerfile` 或使用 CNCF 中 [Cloud Native Buildpacks](https://buildpacks.io/) 的方式從原始碼打包 Container image。

[Google Cloud Next 20' - Hands-on Keynote: Building Trust for Speedy Innovation](https://cloud.withgoogle.com/next/sf/onair?session=SVR227#application-modernization) 中也有場次提到 Google 內部已經使用 Buildpack 的服務

- Google Cloud maintains a set of open source buildpacks for building containers to run on **GKE**，**Anthos**，& **Cloud Run**
- **Cloud Build** native support for buildpacks
- **Cloud Run** direct source deployments w/ buildpacks
- **Cloud Shell** has pack pre-installed. App Engine builds via buildpacks
- **Cloud Functions** builds via buildpacks
- **Cloud Code** deploy to Cloud Run with Buildpacks
- **Skaffold** native support for buildpacks

最大的好處是不需要編寫 `Dockerfile`，Container 相關的安全漏洞也會幫你處理好，更詳細的部份可以參考 [GoogleCloudPlatform/buildpacks: Builders and buildpacks designed to run on Google Cloud's container platforms](https://github.com/GoogleCloudPlatform/buildpacks)

## Cloud Run Button

> Let anyone deploy your GitHub repos to Google Cloud Run with a single click

{{< video src="video/cage1016_gokit-add-cloud-run.mp4" >}}

再來介紹今天的主角 `Cloud Run Button`， [GoogleCloudPlatform/cloud-run-button](https://github.com/GoogleCloudPlatform/cloud-run-button) 可以讓你一鍵部署寫好的 Cloud Run Code. 基本的原理就是 Cloud Run Button 幫你完成 gcloud command 中 Cloud Run 相關的操作，從 source repo clone，build container image，到部署到 Cloud Run 中，非常的方便。

再建立 Container image 的過程中也是可以使用 Buildpack 的方式，也可以在根目錄中配置 `project.toml` 來設定 buildpack 相關的參數。如下

__project.toml__

```toml
[[build.env]]
name = "GOOGLE_BUILDABLE"
value = "cmd/add/main.go"
```

### Cloud Code - Cloud Run

[Cloud Code - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=GoogleCloudTools.cloudcode)

另外 Cloud Code 也有支援 Cloud Run 的整合，Cloud Run Button 是在 Cloud shell 透過 git clone 的方式來獲取原始碼，對於私人專案可能沒有那麼適合，就可以選擇 Cloud Code - Cloud Run，Cloud Code - Cloud Run 一樣有整合 buildpack，同樣適配 `project.toml` 設定參數

{{<image src="img/1.png" src_l="img/1.png">}}

### Demo 

[![Run on Google Cloud](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run?git_repo=https://github.com/cage1016/gokit-add-cloud-run)

https://github.com/cage1016/gokit-add-cloud-run
