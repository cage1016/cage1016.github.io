# Devfest22 Taichung Cloud Workstation Introduction


<!--more-->

{{< slideshare id="253874492" >}}

Slideshare: [DevFest 2022 - Cloud Workstation Introduction TaiChung](https://www.slideshare.net/cagechung/devfest-2022-cloud-workstation-introduction-taichungpdf)

Google 在 2022/10/11 公佈了 [Cloud Workstations](https://cloud.google.com/workstations) preview 版本。在今年 [DevFest Taichung 2022](https://gdg.community.dev/events/details/google-gdg-taichung-presents-devfest-taichung-2022/) 分享了 - Cloud Workstation Introduction. 當中介紹了 Cloud Workstation 怎麼設定、使用場景及 Demo

### 簡單的使用心得

從建立 Cluster, Configuration 及 Workstation 操作上算是方便. 基礎的 Editor [Code-OSS](https://github.com/microsoft/vscode) 可以搭配 [Open VSX Registry](https://open-vsx.org/) 安裝擴充使用。實際操作之後會發現有一些自己常用的 VSCode 擴充套件找不到, 跟 [Visual Studio Marketplace](https://marketplace.visualstudio.com/vscode) 還是有一些區別.

因為整個 Workdation 建立在容器化之後，所以在擴充性上保留了一定的彈性。例如 Cloud Workation 已經整合了 `Code Cloud`，所以開發環境已經提供了相關的指令集如 `gcloud`, `skaffold` 等，不過 `helm` 確沒有包含進去。這時候就可以透過 `Dokcerfile` 來安裝打包含 `helm` 指令的映像檔。以後只要基於該映像檔啟動的容器階會包含 `helm`。這個方便性體現在需要相容開發環下的團隊，在新人加入團隊之後可以省去一定程度時間來準個發開環境

在 EcoSystem 的部份有整合了 Jetbrain 的 IDE，除了需要指定對應的映像檔之外，還需要安裝 [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/)，使用體驗大致上跟本地版本一樣，除了需要經過 Gateway 進行連線之外造成一些功能不支援

- Debug on Kubernetes
- Debug a locally running Cloud Run service

最後目前 Cloud Workstation 是 preview 版本，所以後序的功能會持續更新，但是整體來說如果想到進行開發環境的隔離，或是想要在不同的開發環境之間切換，Cloud Workstation 是一個不錯的選擇
