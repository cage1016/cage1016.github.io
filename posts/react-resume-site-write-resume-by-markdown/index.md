# React Resume Site - Write Resume by Markdown


<!--more-->

在之前找工作的經驗當中，有被要求過只收 104 格式的 Resume，找個工作還要填寫一些沒有相關的欄位，那個年代只好忍一下

## resumeworkshop

{{< youtube M2IrPFMFwx8 >}}

前幾天的 Vscode Channel 剛好有看到教你如果使用 Github dev + Github Page 的方式給自己搭一個簡易的 Resume 網頁 (Full workshop 👉  https://aka.ms/resumeworkshop)。簡單來說就是使用 HTML + CSS 來呈現，搭配 Github Dev + [CodeSwing](https://marketplace.visualstudio.com/items?itemName=codespaces-Contrib.codeswing) Extension 來讓工作流程容易一點

{{<image src="img/final-result.png" alt="final result">}}

Pros:
- 快速
- CodeSwing 加快工作效率

Cons:
- 簡單的模版
- 想增加美觀需要花很多時間調整 HTML + CSS

## React resume site

身為一個工程師，日常使用 Markdown 來記錄各種東西已經是很習慣的方式。在過往經驗找工作必備的 Resume 是否也可以使用 Markdown 來編寫? 把 Resume 弄的簡單乾淨美觀是一件不太容易的事。之前剛好找到一個不錯的開源專案， fork 了一份來修改一下自己所需的部份, [cage1016/react-resume-site: 木及简历|一款用 `Markdown`就能写出好看简历(resume)的在线工具。](https://github.com/cage1016/react-resume-site)

1. 加入的 vscode devcontainer - 開發時可以有效的隔離系統環境，Container 是好物
1. 增加了 `linkedIn` 及 `slideshare` 的 fontawesome Icon
1. 使用 Github Action 透過 Buildpack 來打包成 Container image: [react-resume-site/release.yml](https://github.com/cage1016/react-resume-site/blob/develop/.github/workflows/release.yml)

{{<image src="img/1.jpg" alt="react-resume-site">}}

1. 使用 Markdown 來編寫
1. 可以 embed 圖片。如：Github Readme status [anuraghazra/github-readme-stats: Dynamically generated stats for your github readmes](https://github.com/anuraghazra/github-readme-stats) 很容易豐富頁面
1. 可以 import/export Markdown 檔案
1. 可以輸出 PDF 檔 (很重要的功能)
1. `Docker run` 立即享有 🤘

----

### 知識點

身為一個一直推薦大家使用 [Cloud Native Buildpacks](https://buildpacks.io/) 的人，本次打包 Container Image 當然也要用一下。React resume site 是一個 Node.js 的應用程式。現在大家常用前後端架構分離的條件下，前端自己的 CI/CD Pipeline 最終產物是一堆 HTML + CSS + JS 等等的靜態檔案，上 Container Image 時就可以搭配 Nginx 使用。相較專案本身使用的 `Dockerfile` 單純把整個專案直接放在 Container 上執行的作法是比較少用的

1. 前置作業就是透過 `npm run build` 產出最後的靜態檔案
1. 使用 `gcr.io/paketo-buildpacks/nginx` buildapck 來 Serving 這些靜態檔案
1. Builder 的部份配合 `gcr.io/paketo-buildpacks/nginx`，使用 `paketobuildpacks/builder:base`
1. 準備 [react-resume-site/nginx.conf](https://github.com/cage1016/react-resume-site/blob/develop/nginx.conf) Nginx Config
1. 準備 [react-resume-site/mime.types](https://github.com/cage1016/react-resume-site/blob/develop/mime.types) mime.types
1. 準備 [react-resume-site/project.toml](https://github.com/cage1016/react-resume-site/blob/develop/project.toml) 在打包 Container Image 可以排除一些不需要的檔案 (Optional)


```bash
pack build ghcr.io/cage1016/react-resume-site:0.1.0 --buildpack gcr.io/paketo-buildpacks/nginx --builder paketobuildpacks/builder:base

...
===> DETECTING
paketo-buildpacks/nginx 0.5.2
===> ANALYZING
Previous image with name "ghcr.io/cage1016/react-resume-site:0.1.0" not found
===> RESTORING
===> BUILDING
Paketo Nginx Server Buildpack 0.5.2
  Resolving Nginx Server version
    Candidate version sources (in priority order):
      buildpack.toml -> "1.21.*"

    Selected Nginx Server version (using buildpack.toml): 1.21.4

  Executing build process
    Installing Nginx Server 1.21.4
      Completed in 222ms

  Configuring build environment
    PATH -> "$PATH:/layers/paketo-buildpacks_nginx/nginx/sbin"

  Configuring launch environment
    PATH -> "$PATH:/layers/paketo-buildpacks_nginx/nginx/sbin"

  Writing profile.d/configure.sh
    Calls executable that parses templates in nginx conf
===> EXPORTING
Adding layer 'paketo-buildpacks/nginx:nginx'
Adding 1/1 app layer(s)
Adding layer 'launcher'
Adding layer 'config'
Adding layer 'process-types'
Adding label 'io.buildpacks.lifecycle.metadata'
Adding label 'io.buildpacks.build.metadata'
Adding label 'io.buildpacks.project.metadata'
Setting default process type 'web'
Saving ghcr.io/cage1016/react-resume-site:0.1.0...
*** Images (sha256:cfe7259ffd44b824ab32146427ea06186c8dc2185c44a0125f035572a0125b96):
      ghcr.io/cage1016/react-resume-site:0.1.0
Successfully built image ghcr.io/cage1016/react-resume-site:0.1.0
```

## Usage

1. Pull Container Image
   ```bash
   docker pull ghcr.io/cage1016/react-resume-site:0.1.0
   ```

1. Run Container Image
   ```bash
   docker run --rm --env PORT=8080 -p 8080:8080 ghcr.io/cage1016/react-resume-site:0.1.0
   ```

1. Visit [http://localhost:8080](http://localhost:8080)

## 木及简历

https://www.mujicv.com/

這邊還有更多的模版可以下載
