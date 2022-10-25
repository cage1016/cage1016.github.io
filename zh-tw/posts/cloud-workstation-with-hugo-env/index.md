# Cloud Workstation With Hugo Env


<!--more-->

GCP Cloud Worksation 提供的預配置的基本映像只包含一個最小的環境，包括 IDE、基本的 Linux 終端和語言工具和一個 sshd 伺服器。為了加快特定開發用例的環境設定，我們可以建立自定義容器映像，將這些基本映像擴充套件到預安裝工具和依賴項，並執行自動化指令碼。

**Cloud Worksation 提供以下基本映像檔**
| Image                                                                                    | Description                                                            |
| ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/code-oss:latest          | Cloud Workstations base editor based on [Code-OSS](https://github.com/microsoft/vscode). (Default).           |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/base:latest              | Base image with no IDE installed.                                      |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/clion:latest             | CLion IDE. Accessible only through [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/)                   |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/goland:latest            | GoLand IDE. Accessible only through [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/).                 |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/intellij-ultimate:latest | IntelliJ IDEA Ultimate IDE. Accessible only through [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/). |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/phpstorm:latest          | PhpStorm IDE. Accessible only through [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/).               |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/pycharm:latest           | PyCharm Professional IDE. Accessible only through [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/).   |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/rider:latest             | Rider IDE. Accessible only through [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/).                  |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/rubymine:latest          | RubyMine IDE. Accessible only through [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/).               |
| us-central1-docker.pkg.dev/cloud-workstations-images/predefined/webstorm:latest          | WebStorm IDE. Accessible only through [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/).               |

### Hugo env

> 透過 Worksation 搭建 Hugo 開發環境

- 使用 `us-central1-docker.pkg.dev/cloud-workstations-images/predefined/code-oss:latest` 為基本映像
- 安裝 `Hugo`
- 安裝 [Code-OSS](https://github.com/microsoft/vscode) [Open VSX Registry](https://open-vsx.org/) 擴展
   - [better-toml](https://open-vsx.org/extension/bungcip/better-toml)
   - [vscode-markdownlint](https://open-vsx.org/extension/DavidAnson/vscode-markdownlint)
   - [change-case](https://open-vsx.org/extension/wmaurer/change-case)

```dockerfile
FROM us-central1-docker.pkg.dev/cloud-workstations-images/predefined/code-oss:latest

RUN sudo apt update
# VARIANT can be either 'hugo' for the standard version or 'hugo_extended' for the extended version.
ARG VARIANT=hugo
# VERSION can be either 'latest' or a specific version number
ARG VERSION=latest

# Download Hugo
RUN apt-get update && apt-get install -y ca-certificates openssl git curl && \
    rm -rf /var/lib/apt/lists/* && \
    case ${VERSION} in \
    latest) \
    export VERSION=$(curl -s https://api.github.com/repos/gohugoio/hugo/releases/latest | grep "tag_name" | awk '{print substr($2, 3, length($2)-4)}') ;;\
    esac && \
    echo ${VERSION} && \
    wget -O ${VERSION}.tar.gz https://github.com/gohugoio/hugo/releases/download/v${VERSION}/${VARIANT}_${VERSION}_Linux-64bit.tar.gz && \
    tar xf ${VERSION}.tar.gz && \
    mv hugo /usr/bin/hugo

# Download Extension
RUN wget https://open-vsx.org/api/bungcip/better-toml/0.3.2/file/bungcip.better-toml-0.3.2.vsix && \
unzip bungcip.better-toml-0.3.2.vsix "extension/*" &&\
mv extension /opt/code-oss/extensions/better-toml

RUN wget https://open-vsx.org/api/DavidAnson/vscode-markdownlint/0.48.1/file/DavidAnson.vscode-markdownlint-0.48.1.vsix && \
unzip DavidAnson.vscode-markdownlint-0.48.1.vsix "extension/*" &&\
mv extension /opt/code-oss/extensions/markdownlint

RUN wget https://open-vsx.org/api/wmaurer/change-case/1.0.0/file/wmaurer.change-case-1.0.0.vsix && \
unzip wmaurer.change-case-1.0.0.vsix "extension/*" &&\
mv extension /opt/code-oss/extensions/change-case

# Hugo dev server port
EXPOSE 1313

# [Optional] Uncomment this section to install additional OS packages you may want.
#
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends webp
```

```bash
$ docker build -t cage1016/hugo-evn .
```

### Workstation

1. 在 workstation configuration 設定中的配置客製化後的容器映像檔 `docker.io/cage1016/hugo-env:latest`
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/configuration.jpg" alt="configuration">}}

1. 在容器映像檔為公開時，可以不使用 Service Account
1. 基於客製化的配置建立一台 workstation
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/create-workstation.jpg" alt="create workstation">}}

1. 啟動新建立的 workstation 並連線
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/start.jpg" alt="start workstation">}}

1. 複製 Git 倉庫 `https://github.com/526avijitgupta/gokarna.git`
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/clone.jpg" alt="clone repo">}}

1. 啟動 Hugo

      `$ cd exampleSite/`

      `$ hugo server -D`

      {{<image src="/posts/cloud-workstation-with-hugo-env/img/hugo.jpg" alt="clone repo">}}

1. 透過 gcloud 指令執行 port-forward 來執行本地 Live preview
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/port-forward.jpg" alt="gcloud port forward">}}

1. 訪問 [http://localhost:1313](http://localhost:1313)
1. 即可進行 Hugo 開發

      {{<image src="/posts/cloud-workstation-with-hugo-env/img/placeholder.gif">}}
