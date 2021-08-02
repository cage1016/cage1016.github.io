# Coscup X Ruby Conf Tw 2021 Google Cloud Buildpacks


CNCF 的 Cloud Native Buildpacks 專案已經在 2020 已經從 Sandbox 項目變成成了 Incubation 項目。Cloud Native Buildpacks (CNB) 的出現定義了轉換程式碼至 OCI 的標準，可以讓開發人員專注在本身功能上面，將安全性及依賴套件打包相關的部份交由 CNB 負責。

<!--more-->

Google 也依照了CNB 的規範開源了 Google 版的 Google Cloud Buildpacks，也在 2019 Google Cloud Next 上宣布 Cloud Run (Cloud Run Button), GKE, Anthos, App Engine, Cloud functions 等服務支援使用 Google Cloud Buildpacks。

本場次會介紹 CNB 的發展歷史及 Google 開源的 Google Cloud Buildpacks 剖析與實踐

{{< slideshare id="249900447" >}}

{{< youtube BgdirjYjKWc >}}

之前分享跟 Buildpack 相關的主題
- [Buildpack Tips and Tricks - KaiChu](https://kaichu.io/posts/buildpack-tips-and-tricks/)
- [Build Your Buildpack - KaiChu](https://kaichu.io/posts/build-your-buildpack/)
- [Github Assets Cnb - KaiChu](https://kaichu.io/posts/github-assets-cnb/)
- [Pack 0.19.0 Solved Invalid Cross Device Link at Google Cloud Build - KaiChu](https://kaichu.io/posts/pack-solved-invalid-cross-device-link-at-google-cloud-build/)

總的來說

1. Buildpack 的出現，提供了一個從 Source to container image 的方式，讓開發人員可以從 `Dockerfile` 中解放，專注在商業邏輯的部份，將 container image 的安全性、每一層怎麼堆疊等部份交給 Buildpack 所規範的方式去作。`Dockerfile` 在某些場景還是很好用，在適合的場景時機引入 Buildpack 則可以讓工作流程簡化
1. [Google Cloud Buildpacks](https://github.com/GoogleCloudPlatform/buildpacks) 是一個 100% 相容 [Cloud Native Buildpacks](https://buildpacks.io/) 的一個實作。類似有實作的還有 Paketo 及 Heroku 所提供的版本，當然各家實作出來的 Builder 除了相容規範之外還會加上自家獨有的東西，如 Google Cloud Buildpacks 就增加了 Google Cloud Platfrom 中特有的 Cloud Function
1. 入級: 每一個 Builder 的參數不盡相同，使用前需參考使用說明。Google Cloud Buildpacks 已經支持 Cloud Run, GKE, Anthos, App Engine, Cloud Function。言話部份也有 Go, Node.js, Python, Java, Net Core 等，在開發工具鍊中也有 skaffold, pack, tekton, kpack 也可以搭配 gcloud 及 Google Cloud Build 一起使用。
1. 初級: 依照 [Cloud Native Buildpacks](https://buildpacks.io/) 的規範實作自己的 Buildpack (shell 版本)
1. 中級: 使用第三方工具來建構 Buildpack, 如 [paketo-buildpacks/packit](https://github.com/paketo-buildpacks/packit)。在閱讀 [Google Cloud Buildpacks](https://github.com/GoogleCloudPlatform/buildpacks) 的開收源始碼後，你會發現這個專案也是使用同樣的方式來建構出 Google 版的 Builpack，不過用的是 Google 自己寫 Library，最終透過 Bazel 打包成相容 Cloud Native Buildpacks 的格式。
