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

身為一個工程師，日常使用 Markdown 來記錄各種東西已經是很習慣的方式，剛好找到一個不錯的開源專案。自己就 fork 了一份來修改一下 [cage1016/react-resume-site: 木及简历|一款用 `Markdown`就能写出好看简历(resume)的在线工具。](https://github.com/cage1016/react-resume-site)

1. 加入的 vscode devcontainer - 開發時可以有效的隔離系統環境，Container 是好物
1. 增加了 `linkedIn` 及 `slideshare` 的 fontawesome Icon
1. 使用 Github Action 透過 Buildpack 來打包成 Container image: [react-resume-site/release.yml](https://github.com/cage1016/react-resume-site/blob/develop/.github/workflows/release.yml)

{{<image src="img/1.jpg" alt="react-resume-site">}}

1. 使用 Markdown 來編寫
1. 可以 embed 圖片。如：Github Readme status [anuraghazra/github-readme-stats: Dynamically generated stats for your github readmes](https://github.com/anuraghazra/github-readme-stats) 很容易豐富頁面
1. 可以 import/export Markdown 檔案
1. 可以輸出 PDF 檔 (很重要的功能)
1. `Docker run` 立即享有 🤘

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
