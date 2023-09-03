# Golang Alfred Workflow Generator Ak


<!--more-->

作為一名 Gopher 同時也是 Alfred workflow 的重度使用者，在日常中也常常編寫 Alfred workflow 來完善加速工作流程。由於 Golang 語言的特性，當我們使用 Golang 語言來發開 Alfred workflow 時會遇到幾個問題

1. Golang 語言是 Binary 的語言，因此在開發時需要先編譯成 Binary 才能使用，在發佈時也需要將 Binary 一起打包到 `.alfreworflow` 中，這邊會遇到 Binary 安全性問題，所以我們需要對我們的 Binary 進行 Code Signing 及 Notarization 才能通過 macOS 的 Gatekeeper 檢查。
1. Alfred workflow 的開發目錄需要對 `Alfred.alfredpreferences` 進行 sybolic link，我們的 workflow 才有辦法跟 Alfred 進行溝通。

早期在開發 Alfred workflow 時，使用 Python 的[workflow-install.py](https://gist.github.com/cage1016/a633149f4672fb40ecb4e16e04664ff7) 來進行相關的 Alfred 功能連動，但是在使用上就是不很方便，因此我們需要一個更好的解決方案。所以就有了這個專案 [cage1016/ak: A generator for golang alfred workflow that helps you create boilerplate code.](https://github.com/cage1016/ak)。

## 功能

1. 快速建立 Golang Alfred workflow 專案的 boilerplate code
    - 使用 [deanishe/awgo](https://github.com/deanishe/awgo) 及 [spf13/cobra](https://github.com/spf13/cobra) 作為 Alfred workflow 的開發框架
    - 建立二組 Github Action release 流程 (自動更新版及 Alfred Gallery 發佈兼容版)，並且支援 Code Sign 及 Notarization
    - 建立 Makefile 來簡化開發流程
    - 建立 LICENSE
2. Workflow 開發環境的管理
    - 透過 `make link` 來將 workflow 的開發目錄對 `Alfred.alfredpreferences` 進行 symbolic link
    - 透過 `make unlink` 來將 workflow 的開發目錄對 `Alfred.alfredpreferences` 進行 unlink
    - 透過 `make build` 來將 workflow 的開發目錄對 `Alfred.alfredpreferences` 進行 build
    - 透過 `make info` 來顯示 workflow 的開發目錄對 `Alfred.alfredpreferences` 的資訊
    - 透過 `make pack` 來將 workflow 的開發目錄對 `Alfred.alfredpreferences` 進行打包
3. 支援 `arm64` & `amd64`

## 安裝

使用 Go 安裝

```bash
go install github.com/cage1016/ak@latest
```

## 使用方式

1. 建立專案目錄
1. 執行 `ak` 來建立 `ak.json` 檔案及設定後續的專案資訊
   ```json
   {
       "go_mod_package": "github.com/xxx/ak-test",
       "workflow": {
           "folder": ".workflow",
           "name": "Ak Test",
           "description": "",
           "category_comment": "category: Tools, Internet, Productivity, Uncategorised",
           "category": "",
           "bundle_id": "com.xxx.aktest",
           "created_by": "",
           "web_address": "https://github.com/xxx/ak-test",
           "version": "0.1.0"
       },
       "update": {
           "github_repo": "https://github.com/xxx/ak-test"
       },
       "license": {
           "type_comment": "support license https://github.com/nishanths/license",
           "type": "mit",
           "year": "",
           "name": ""
       },
       "gon": {
           "application_identity": ""
       }
   }
   ```
1. 執行 `ak init` 來建立專案的 boilerplate code
    - `.workflow` 目錄為 Alfred workflow 的開發目錄
    - `.gitingore` 為專案的 git 忽略檔案
    - `.env` 為專案的環境變數檔案
1. 執行 `ak new cmd` 來建立基於 Cobra CLI 的 command 
1. 執行 `ak add ga` 來建立 Github Action 的 release 流程
    - `-s` 啟用 Code Sign 及 Notarization 功能
    - `-c` 啟用 Go test Codecov 發佈功能
1. 執行 `ak add mf` 來建立 Makefile 的開發管理工具
1. 執行 `ak add l` 來建立 LICENSE 檔案

基本上到這邊就可以開始進行開發了，如果有需要可以透過 `cobra add` 來增加更多的 command 來完善 workflow 的功能。[ak/_example](https://github.com/cage1016/ak/tree/master/_example) 這邊有一個完整的範例專案可以參考。

## 開發流程

1. 執行 `go run main.go` 來執行 workflow，會提供下載 go mod 有使用到的套件並使用 `go get` 來安裝
1. 執行 `make build` 來將 workflow 的開發目錄對 `Alfred.alfredpreferences` 進行 build，這個時會將編譯好的 Binary `exe` 放到 `.workflow` 目錄下
1. 執行 `make link` 來將 workflow 的開發目錄對 `Alfred.alfredpreferences` 進行 symbolic link

當完成 Alfred workflow symbolic link 後，我們就可以在 Alfred workflow 看到我們的 workflow ，這個時候我們就可以開始進行開發了。

## 發佈流程

1. 執行 `make pack` 來將 workflow 的開發目錄進行本地打包，這個時候會將 workflow 打包成 `.alfredworkflow` 檔案
1. `ak add ga -s -c` 會產生二組 Github Action release 流程 (自動更新版及 Alfred Gallery 發佈兼容版)，並且支援 Code Sign 及 Notarization. 這個時候我們就可以透過 Github Action 來進行自動化的發佈流程。Code Sign 及 Notarization 怎麼使用請查看 [Golang Base Alfred Code Sign and Notarize - KaiChu](https://kaichu.io/zh-tw/posts/golang-base-alfred-code-sign-and-notarize/)

## 結論

這個專案是我在開發 Alfred workflow 時，為了解決開發流程及發佈流程的問題而開發的，希望這個專案可以幫助到大家，如果有任何問題歡迎在 [cage1016/ak: A generator for golang alfred workflow that helps you create boilerplate code.](https://github.com/cage1016/ak) 提出 issue。

## 原始碼

[cage1016/ak: A generator for golang alfred workflow that helps you create boilerplate code.](https://github.com/cage1016/ak)
