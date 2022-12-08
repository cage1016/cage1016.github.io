# Devfest22 Taipei Skaffold2 Deep Dive


<!--more-->

{{< slideshare id="254778592" >}}

Slideshare: [DevFest 2022 - Skaffold 2 Deep Dive Taipei.pdf](https://www.slideshare.net/cagechung/devfest-2022-skaffold-2-deep-dive-taipeipdf)

簡單回顧一下歷年在 Devfest 上的分享

- [Devfest19 Build Go Kit Microservices at Kubernetes With Ease](https://kaichu.io/zh-tw/posts/devfest19-build-go-kit-microservices-at-kubernetes-with-ease/)
- [Devfest20 How to Debug Microservices on Kubernetes as a Pros](https://kaichu.io/zh-tw/posts/devfest20-how-to-debug-microservices-on-kubernetes-as-a-pros/)
- [Coscup X Ruby Conf Tw 2021 Google Cloud Buildpacks](https://kaichu.io/zh-tw/posts/coscup-x-ruby-conf-tw-2021-google-cloud-buildpacks/) - COSCUP
- [Devfest21 Taipei Artifact Registry Introduction](https://kaichu.io/zh-tw/posts/devfest21-taipei-artifact-registry-introduction/)
- [Devfest22 Taichung Cloud Workstation Introduction](https://kaichu.io/zh-tw/posts/devfest22-taichung-cloud-workstation-introduction/)

2022 Devfest 台北場其實也延續了 2021 年的主題，也就是 Cloud Native 相關在 Container 上的技術分享, 這次我們來聊聊 Skaffold 2 的新特性, 一個對於開發者來說非常友善的工具，如果你是一個 Kubernetes 應用開發者，那麼 Skaffold 2 將會是你的好幫手. 

雖說是 Skaffold 2 deep dive 的場子。詢問了參與的會眾，結果大部份的人都沒有聽過 Skaffold 這個名詞，提問的問題反而聚焦在 Skaffold 2 底層運用的技術，例如在建立容器時透過 buildpack 就不需要編寫 `Dockerfile` 的使用場景、適用性及進入門檻等等，還有 Skaffold 在 CI/CD 角色伴演的等等，這些問題其實都是很好的問題

{{<image src="img/005.jpg" alt="Skaffold">}}

Skaffold 是一個專注於開發者的工具，它的目標是讓開發者可以在 Kubernetes 上快速的開發、測試、部署應用程式。在提供的 Pipeline Stages 中都可以依照自己的喜好、適用的場景來選擇對應的選項，彈性非常高。例如我個人喜歡使用 buildpack 來取代編寫 `Dockerfile`，如果面對的是一個非主流的場景怎麼辦? 這時候還是依然可以選擇使用 `Dockerfile` 來建立容器，完全自主撑控.

## Timeline
{{<image src="img/placeholder.jpg" alt="timeline">}}

從這一張圖我們可以看到 Skaffold 的發展其實非常的快速

1. 2018/3/6 發佈了 `v0.1.0` 經過 46 次的發佈，於 2019/12/21 將版本推上了 `v1.1.0`
1. `v1.1.0` 釋出之後，在 3 年中更是迭代了 72 次之後，於 2022/10/21 將版本推上了 `v.2.0.0`

## Highlights & New Features
- 💻 Support for deploying to ARM, X86 or Multi-Arch K8s clusters from your x86 or ARM machine
- 👟 New Cloud Run Deployer brings the power of Skaffold to Google Clouds serverless container runtime
- 📜 Skaffold render phase has been split from deploy phase providing increased granularity of control for GitOps workflows
- 🚦New Skaffold verify phase enables improved testing capabilities making Skaffold even better as a CI/CD tool
- ⚙ Tighter integration with kpt lets you more dynamically manage large amounts of configuration and keep it in sync

[Release v2.0.0 Release · GoogleContainerTools/skaffold](https://github.com/GoogleContainerTools/skaffold/releases/tag/v2.0.0)

Skaffold V2 擴充套件了 Skaffold 支援的平臺和架構，引入了 Cloud Run 作為支援的部署器，現在支援從 ARM 和 x86 架構構建並部署到 ARM 和 x86 架構。Skaffold V2 還提供了對 CI/CD 和 GitOps 工作流程的增強支援，引入了 skaffold render、verify 和 kpt 整合。最重要的是，所有現有的 Skaffold 配置都與 Skaffold V2 完全相容，從 V1 升級就像執行 skaffold fix 一樣容易

## Skaffold Pipeline Stages

{{<image src="img/021.jpg" alt="Architecture">}}
{{<image src="img/024.jpg" alt="Skaffold Pipeline Stages">}}
{{<image src="img/022.jpg" alt="Architecture - Cloud Build / Helm">}}

Skaffold Architecture / Pipeline Stages 是 Skaffold 的核心，它提供了一個 Pipeline 的架構, 圍繞在 `Build`, `Test`, `Tag`, `Deploy` 幾個面向. 熟悉每一個 Pipeline Stage 的用途及運用，對於 Skaffold 的使用會更加的熟悉，也能夠更加的自在的運用 Skaffold 來開發、測試、部署應用程式.

## Tips & Tricks

{{<image src="img/007.jpg" alt="Install">}}

Skaffold 也有被整合到即有的 IDE 上 (VSCode / IntellJ) / Google Cloud Shell / Cloud Workstation，一起使用方便快速.

{{< admonition warning "Code Code" true >}}
Cloud Code (**v1.20.4**, Nov 2022) not support Skaffold 2 yet, current build-in skaffold version is **v1.39.3**
{{< /admonition >}}

其他詳細的內容請參閱 [DevFest 2022 - Skaffold 2 Deep Dive Taipei.pdf](https://www.slideshare.net/cagechung/devfest-2022-skaffold-2-deep-dive-taipeipdf)
