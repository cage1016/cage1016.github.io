# [筆記] docker 1.8.2 rc


<!--more-->

在 Mac os x 開發 docker，因為 Mac 無法原生支援 Docker，所以在 docker 1.7 以前的版上就必需透過 `boot2docker`，`boot2docker`
會在本機的 VirtualBox 上安裝一個虛擬機。Docker client 機乎是接近原生的狀態跑在 Mac 上，只不過 Docker 的 server 是跑在 `boot2docker` 的虛擬機中.

{{<image src="img/docker_on_linux_macosx.jpg">}}
picture: https://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide

XD

```shell
WARNING: The 'boot2docker' command line interface is officially deprecated.

Please switch to Docker Machine (https://docs.docker.com/machine/) ASAP.

Docker Toolbox (https://docker.com/toolbox) is the recommended install method.
```

{{<image src="img/mac-page-two.png">}}
picture: https://docs.docker.com/installation/mac/#installation

docker 社群開發的速度非常的快速，而 docker 1.8 之後就有了比較大的變化。安裝的工具也發生了變化，
多了 [Docker Toolbox](http://docs.docker.com/mac/step_one/#step-2-install-docker-toolbox)，官方的安裝步驟 for Mac [點我](http://docs.docker.com/mac/step_one/)

安裝完了 Docker Toolbox 之後，可以直接在 `Launchpad` 上尋找 `Docker Quickstart Terminal`，執行之後終端機會進行 VirtualBox 初始化，
並建立 default 的虛擬機(在 1.7 版時則是利用 `boot2docker` 來建立虛擬機，並會在 VirtualBox 中建立一個名為 `boot2docker-vm` 的虛擬機)。

```shell
$ bash --login '/Applications/Docker/Docker Quickstart Terminal.app/Contents/Resources/Scripts/start.sh'
Machine default already exists in VirtualBox.
Starting machine default...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
Setting environment variables for machine default...


                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/


docker is configured to use the default machine with IP 192.168.99.100
For help getting started, check out the docs at https://docs.docker.com
```

接著利用 `docker-machine` 來查詢目前那些 Docker host

```shell
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM
default   *        virtualbox   Running   tcp://192.168.99.100:2376
dev                virtualbox   Stopped
```

## Simple Demo

接下來利用簡單的步驟來試範在 Docker 上跑一個 Nginx 並透過設定對外開放 port 讓外部可以直接存取到 Nginx 的頁面

```shell
# 確認 docker-machine env 狀態 (我們以預設的 Docker host - default)
$ docker-machine env default

# 必要時重置一下 shell 環境
# eval "$(docker-machine env default)"

# 建立資料夾
$ mkdir web && cd web

# 執行一個 container
# -d daemon: 設定 container 跑在背景執行
# -p expose port: 設定 container 的 port 與 docker host 的 port mapping, Nginx 預設 port 為 80
# --name 指定一個名子給 container
$ docker run -d -p 80:80 --name web nginx

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
e6644e903590        nginx               "nginx -g 'daemon off"   3 seconds ago       Up 4 seconds        0.0.0.0:80->80/tcp, 443/tcp   web

# 訪問 Nginx
$ curl http://$(docker-machine ip default):80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Tips

```shell
$ docker info
Get http:///var/run/docker.sock/v1.20/info: dial unix /var/run/docker.sock: no such file or directory.
* Are you trying to connect to a TLS-enabled daemon without TLS?
* Is your docker daemon up and running?
```

如果遇到這個問題，只有重置一下 `docker-machine` 即可，Docker 還很貼心的告訴你如何重置 shell `eval "$(docker-machine env default)"`

```shell
$ docker-machine env default
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/cage/.docker/machine/machines/default"
export DOCKER_MACHINE_NAME="default"
# Run this command to configure your shell:
# eval "$(docker-machine env default)"
```

# 相關參考資料

1. [Mac OS下的boot2docker | 最完整的Docker聖經 - Docker原理圖解及全環境安裝](https://joshhu.gitbooks.io/docker_theory_install/content/DockerBible/mac_osboot2docker.html)
1. [在 Mac OS X 系统里使用 Docker - 品雪其寒 - 博客频道 - CSDN.NET](http://blog.csdn.net/pinxue/article/details/45370579)
1. [Installation on Mac OS X](https://docs.docker.com/installation/mac/#installation)

