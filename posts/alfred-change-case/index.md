# Alfred Change Case


<!--more-->

## Alfred 4 for Mac
> Alfred is an award-winning app for macOS which boosts your efficiency with hotkeys, keywords, text expansion and more. Search your Mac and the web, and be more productive with custom actions to control your Mac.

[Alfred - Productivity App for macOS](https://www.alfredapp.com/)
 
{{<image src="img/1.png" alt="alfred">}}

Alfred 是一個在 Mac 上面增加生產力的工具，舉凡工程師常用的 Github、Google Drive 搜尋到 LeetCode Search 等原都可以在 Alfred 完成，簡直就是太方便了。自己也有曾經想要開發自己的 Alfred 應用，直到遇到 [deanishe/awgo: Go library for Alfred 3 + 4 workflows](https://github.com/deanishe/awgo) 之後就像撿到寶一樣，作為一個 Gopher 也找到適合的工具來開發自己的 Alfred 應用。

## Change case alfred workflow

> This is change case tools inspired by [change-case - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=wmaurer.change-case) and wrapper by [ku/go-change-case](https://github.com/ku/go-change-case). Quickly change the case of a string or latest clipboard string and copy it to the clipboard.


### Select Change Case options

{{<image src="img/1.gif" alt="Select Change Case options">}}

### Change Case by specific option directly

{{<image src="img/2.gif" alt="Change Case by specific option directly">}}

{{<image src="img/3.png" alt="Change Case by specific option directly">}}

由於自己平常對於文串轉換的需求不小，在 vscode 中可以依賴 [change-case - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=wmaurer.change-case) 來完成需求；不過若在 vscode 環境之外就沒有那麼方便，也就是這一次 slide project - [cage1016/alfred-change-case](https://github.com/cage1016/alfred-change-case) 的由來

### alfred-change-case

{{<image src="img/placeholder.png" alt="alfred">}}

本次有使用到下面的知識點

- [deanishe/awgo: Go library for Alfred 3 + 4 workflows](https://github.com/deanishe/awgo): Go library for Alfred 3 + 4 workflows
- [ku/go-change-case](https://github.com/ku/go-change-case): a golang port of npm package change-case
- `pyenv` + `virtualenv` + `pip` 準備 Python 環境 + `workflow-install.py` 來將 `.workflow` 所需的檔案動態安裝至 Alfred 中，在 vscode 的環境中搭配 vscode 的 tasks 來執行非常方便
  - go build
  - prepare-info.plist
  - install
- Github action 使用 `runs-on: macOS-latest` 來打包整個 `.alfredworkflow` 並上傳至 Github release asset 中，建立一個 Release 就可以完成發佈的動作

## 心得

這一次的 [cage1016/alfred-change-case](https://github.com/cage1016/alfred-change-case) 的字串轉換基本封裝 [ku/go-change-case](https://github.com/ku/go-change-case) 為基礎轉換成 Alfred 的 workflow。在 [deanishe/awgo: Go library for Alfred 3 + 4 workflows](https://github.com/deanishe/awgo) 上單純進行簡單的流程操作，作為一個 Gopher 來說是覺得開發過程很流暢，這一次的開發經驗可以很快的複製到其他的 Alfred workflow 專案

- Source code: [cage1016/alfred-change-case](https://github.com/cage1016/alfred-change-case)
- Download: [change-case-1.0.0.alfredworkflow](https://github.com/cage1016/alfred-change-case/releases/download/v1.0.0/change-case-1.0.0.alfredworkflow)
