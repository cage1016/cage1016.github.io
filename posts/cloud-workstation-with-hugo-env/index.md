# Cloud Workstation With Hugo Env


<!--more-->

The preconfigured base images provided by Cloud Workstations contain only a minimal environment with IDE, basic Linux terminal and language tools and a sshd server. To expedite the environment setup of specific development use cases, you can create custom container images that extend these base images to pre-install tools and dependencies and that run automation scripts.

**Base container image provided by Cloud Worksation**
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

> Build Hugo development environment by GCP cloud workstation

- Use `us-central1-docker.pkg.dev/cloud-workstations-images/predefined/code-oss:latest` base container image
- Install `Hugo`
- Install [Code-OSS](https://github.com/microsoft/vscode) [Open VSX Registry](https://open-vsx.org/) extenions
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

1. Setup workation base container image at workstation configuration `docker.io/cage1016/hugo-env:latest`
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/configuration.jpg" alt="configuration">}}

1. We could let Service Account field empty if base container image is public.
1. Create workstation by previous configuration
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/create-workstation.jpg" alt="create workstation">}}

1. Start workstaion and luanch it.
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/start.jpg" alt="start workstation">}}

1. Clone git repo from `https://github.com/526avijitgupta/gokarna.git`
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/clone.jpg" alt="clone repo">}}

1. Develop Hugo

      `$ cd exampleSite/`

      `$ hugo server -D`

      {{<image src="/posts/cloud-workstation-with-hugo-env/img/hugo.jpg" alt="clone repo">}}

1. We need to enable gcloud cli port-forward for local live preview
      {{<image src="/posts/cloud-workstation-with-hugo-env/img/port-forward.jpg" alt="gcloud port forward">}}

1. Visit [http://localhost:1313](http://localhost:1313)
1. Enjoying Hugo development.

      {{<image src="/posts/cloud-workstation-with-hugo-env/img/placeholder.gif">}}
