# Devfest21 Taipei Artifact Registry Introduction


Artifact Registry 可以說是 Container Registry 的進化版，可讓貴機構集中管理容器映像檔和語言套件 (例如 Maven 和 npm)。Artifact Registry 與 Google Cloud 的工具和執行階段全方位整合，並且支援原生構件通訊協定，讓您輕鬆與持續整合/持續推送軟體更新 (CI/CD) 工具整合，進而設定自動化管道。

<!--more-->

{{< slideshare id="250751041" >}}

Slideshare [Devfest 2021' - Artifact Registry Introduction (Taipei)](https://www.slideshare.net/cagechung/devfest-2021-artifact-registry-introduction-taipei)

## 功能性

作為 Container Registry 下一代通用解決方案的繼任者 Artifact Registry 除了延續原有的功能之外增加了對 Package 類型的支持

- Container images: [Docker](https://cloud.google.com/artifact-registry/docs/docker), [Helm](https://cloud.google.com/artifact-registry/docs/helm)
- Language packages: [Java](https://cloud.google.com/artifact-registry/docs/java), [Node.js](https://cloud.google.com/artifact-registry/docs/nodejs), [Python](https://cloud.google.com/artifact-registry/docs/python)
- OS packages (Preview): [Debian](https://cloud.google.com/artifact-registry/docs/os-packages/debian), [RPM](https://cloud.google.com/artifact-registry/docs/os-packages/rpm)

Aritfact Registry 在 Container image 的支持了 Docker V2, OCI Image Format 二種格式。在 Aritfact Registry 實作了 [OCI Specification](https://github.com/opencontainers/distribution-spec/blob/master/spec.md) + Helm 3 支持了 [chart packages in OCI format](https://helm.sh/docs/topics/registries/) 的功能，也因為我們可以把 Helm chart 儲存在 Artifact registry 之中，這個也解決了上一代，我們的 Container image 可以放在 Container Registry 而我們的私有的 Helm chart 卻需要使用 Helm + GCS plugin 來搭配使用

{{<image src="img/1.png" caption="Container Registry + Helm + GCS plugin">}}

{{<image src="img/2.png" caption="Artifact Registry">}}

{{< admonition type=tip title="This is a tip" open=true >}}
想把 Helm chart 儲存在 Artifact Registry 時，需要注意 Helm 的版本，Helm 在 **[Release Helm 3.7.0 · helm/helm](https://github.com/helm/helm/releases/tag/v3.7.0)** 把對於 oci experimental 相關的 subcommand 移除了，所以 3.7.0 以上的版本就必需先透過 `helm package` 產出 `tgz` 檔案，再將 `tgz` 推送至 Artifact Registry
{{< /admonition >}}

## 靈活性

{{<image src="img/3.png" caption="Multiple repositories">}}

Artifact Registry 增加了在同一個 GCP Project 可以建立出個 `Repository`，可以分別對應到不同的屬性 (Container images / Language packages / OS packages)，單一 `Repository` 可以指定至特別的 `Zone` 或是 `Region`，其中一個特點是被指派的每一個 `Region` 都相隔 100 英里以上。好處是有異地備援的概念，壞處會反應在 Billing 上面，跨 `Region` Egress 是會收費的

總之多了搭配的可能性，大家可以依照使用場景來選擇自己適合的組合

## 安全性

{{<image src="img/4.png" caption="IAM Role">}}

安全性一直是很重要的題目。在上一代想把 Helm Chart 部署在 GCP 平台上就會使用 Helm + GCS plugin。在這種組合之下權限控管就會變成 GCS 檔案的儲取權

- roles/storage.objectViewer
- roles/storage.legacyBucketWriter
- roles/storage.admin

而 Aritfact Registry 在安全性管控這一塊提供更細粒度的彈性，讓我們可以跟據使用場景指派適合的權限來降低安全風險

- Project Owner
  - roles/artifactregistry.repoAdmin
  - roles/artifactregistry.admin
- Project Editor
  - roles/artifactregistry.writer
- Project Viewer
  - roles/artifactregistry.reader

## Summary

{{<image src="img/5.png" caption="Summary">}}

{{< admonition type=info title="Info" open=true >}}
Artifact Registry is the recommended service for managing container images. Container Registry is still supported but will only receive critical security fixes
{{< /admonition >}}

Artifact Registry 作為 Container Registry 的進化版，除了保留 Container Registry 原有的功能之外，我們可以看到從 `功能性` `靈活性` `安全性` 上面的加強。而官方也有聲名, Container Registry 除了安全性修復之外不會再有新功能的計劃，說明文件也有遷移的教學文件可以參考

最後還是再強調，大家還是需要針對自己的使用場景來選擇組合，才會用的開心，看到 Billing 也開心
