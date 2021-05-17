# Atom sync-settings


<!--more-->

[Atom](https://goo.gl/LGBVzh) 是一個 GitHub 開發的免費、開放原始碼編輯器，目前有非常多的擴充工具可以安裝(**4,913** packages)，也有非常多的主題佈景可以安裝(**1,645** themes)，彈性非常的高，你可以打造成自己熟悉的開發環境。

{{<image src="img/atom-sync-settings-02.jpg">}}


當你好不容易打造好自己熟悉的開發環境時，如果要換電腦時，重新還原熟悉的開發環境是需要花一點功夫。以下是手動的方式

備份機上的清單到一個檔案
```shell
$ apm list --installed --bare | grep '^[^@]\+' -o > my_atom_packages.txt
```

在另外一台機器上使用 `apm` 進行批次安裝的動作
```shell
$ apm install --packages-file my_atom_packages.txt
```

上述的動作雖然可以幫你快速的解決安裝擴充工具的問題，不過設定檔得自已再進行設定，還是有一點麻煩

## [atom-community/sync-settings](https://github.com/atom-community/sync-settings)

> Synchronize all your settings and packages across atom instances

一個超極方便的擴充工具來解決你 Atom 上擴充工具的同步及其設定，**sync-settings** 是透過 Github [Gist](https://gist.github.com/) 來幫你備份擴充工具的清單即設定，所以在 sync-settings 的設定中你必需填入

  - **Personal Access Token** (一組適合特殊需求專給特定程式介接存取 github 資源用:這就是 sync-settings)
  - **Gist Id** (你的 Gist id)

### 建立 Personal Access Token

{{<image src="img/atom-sync-settings-03.jpg">}}

打開 [Github: Personal Access Tokens](https://github.com/settings/tokens/new) > 點選 gist > Generate token > 複製並填入 Atom:sync-settings 中

### 建立 Gist

打開 [Create a new Gist](https://gist.github.com/) 填上說明 > Create secret gist > 複製 Gist Id 並填入 Atom:sync-settings 中

{{<image src="img/atom-sync-settings-04.jpg">}}
{{<image src="img/atom-sync-settings-05.jpg">}}

### Sync Backup

在填入 Personal Access Token 及 Gist Id 後，就可以進行同步的動作 `Shit + command + P` (Mac), `Ctrl + Shit + P` (Windows)

{{<image src="img/atom-sync-settings-06.jpg">}}
{{<image src="img/atom-sync-settings-07.jpg">}}

### Sync restore

在其他電腦上裝好 Atom > 安裝 sync-settings > 進行 Sync: restore 的動作 `Shit + command + P` (Mac), `Ctrl + Shit + P` (Windows)

{{<image src="img/atom-sync-settings-08.png">}}
{{<image src="img/atom-sync-settings-09.png">}}

最後分享一下我的 Atom 擴充工具清單

```shell
Sublime-Style-Column-Selection
atom-alignment
atom-beautify
atom-ternjs
auto-update-packages
autocomplete-go
autocomplete-plus
builder-go
change-case
cobalt2-syntax
color-picker
docblockr
emmet
environment
file-icons
fold-functions
go-config
go-debug
go-get
go-plus
go-rename
godoc
gofmt
gometalinter-linter
gorename
highlight-selected
imdone-atom
jshint
language-c
language-docker
language-gitignore
language-protobuf
language-shellscript
linter
linter-eslint
linter-js-standard
linter-jscs
linter-jshint
mocha-test-runner
navigator-godef
open-recent
package-sync
pigments
project-manager
react
regex-railroad-diagram
rst-preview-pandoc
set-syntax
seti-syntax
seti-ui
sync-settings
terminal-plus
tester-go
todo-show
tool-bar
tool-bar-main
trailing-spaces
tree-view
```

