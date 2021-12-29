# Build Jmeter Docker With Plugins


<!--more-->

最近工作上有使用 [Apache JMeter](https://jmeter.apache.org/) 的需求，使用的情境很簡單，在硬體有支援的情況下進行靜態 website 的壓力測試，所以記錄一下最近的工作內容

- static website: [cage1016/static-website-example: Static website to use with Cloud Academy labs](https://github.com/cage1016/static-website-example)
- Nginx: `nginx:1.20.2-alpine`
- Jmeter with Plugins: [cage1016/docker-jmeter-s390x](https://github.com/cage1016/docker-jmeter-s390x)
- Jmeter Test Plan
- Usage

## Static website
> To be used with Cloud Academy labs.

{{<image src="./img/static_website.jpg">}}

這是一個取自 Cloud Academy labs 建立的靜態網頁，剛好可以拿來作為 Jmeter 壓力測試的網頁，搭配 Nginx 一起使用

_Dockerfile_

```dockefile
FROM nginx:1.20.2-alpine

COPY health /usr/share/nginx/html
COPY default.conf /etc/nginx/conf.d/default.conf
COPY assets /usr/share/nginx/html/assets
COPY error /usr/share/nginx/html/error
COPY images /usr/share/nginx/html/images
COPY index.html /usr/share/nginx/html
```

```sh
docker build -t ghcr.io/cage1016/nginx-website-gz:0.1.0 .
```

**Container images**

- [ghcr.io/cage1016/nginx-website:0.1.0](https://github.com/cage1016/static-website-example/pkgs/container/nginx-website)
- [ghcr.io/cage1016/nginx-website-gz:0.1.0](https://github.com/cage1016/static-website-example/pkgs/container/nginx-website-gz)
- [ghcr.io/cage1016/nginx-website-s390x:0.1.0](https://github.com/cage1016/static-website-example/pkgs/container/nginx-website-s390x)
- [ghcr.io/cage1016/nginx-website-gz-s390x:0.1.0](https://github.com/cage1016/static-website-example/pkgs/container/nginx-website-gz-s390x)

## Jmeter

在進行 Jmeter 部份時，考慮到測試環境以 command line 工具進行為主比較方便，所以選擇了 Jmeter 的 Docker 版本 [justb4/jmeter](https://hub.docker.com/r/justb4/jmeter/)，不過遇到這個版本不支援 IBM s390x 的架構。另外在 [azure devops - Using JMeter plugins with justb4/jmeter Docker image results in error - Stack Overflow](https://stackoverflow.com/questions/67911367/using-jmeter-plugins-with-justb4-jmeter-docker-image-results-in-error) 有看到 justb4/jmeter 使用 Plugin 報出一些問題，所以就自己包啦

- 找到相關的 Dockerfile
- 下載 Jmeter 及額外所需的 Plugins

**找到相關的 Dockerfile**

感謝 Google / Github。[linux-on-ibm-z/dockerfile-examples](https://github.com/linux-on-ibm-z/dockerfile-examples/blob/master/Archived/ApacheJMeter/Dockerfile) 雖然沒有在維護了，不過可以讓我們大至知道需要那些部件，基本架構可以使用 (Base image、openjdk 等等)

**下載 Jmeter 及額外所需的 Plugins**

所需的 Jmeter 及額外所需的 Plugins 都可以透過 [Central Repository: kg/apc](https://repo1.maven.org/maven2/kg/apc/) 這個網站得

_Dockerfile_

```dockerfile
...
# Download from git and build
RUN mkdir -p /tmp/dependencies \
 && wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-$JMETER_VER.tgz -O /tmp/dependencies/apache-jmeter-$JMETER_VER.tgz \
 && mkdir -p /opt \
 && tar -xvzf /tmp/dependencies/apache-jmeter-$JMETER_VER.tgz -C /opt \
 ;

# plugins
RUN wget ${JMETER_PLUGINS_DOWNLOAD_URL}/jmeter-plugins-manager/${JMETER_PLUGINS_MANAGER_VERSION}/jmeter-plugins-manager-${JMETER_PLUGINS_MANAGER_VERSION}.jar -O $JMETER_HOME/lib/ext/jmeter-plugins-manager-${JMETER_PLUGINS_MANAGER_VERSION}.jar \
 && wget ${JMETER_PLUGINS_DOWNLOAD_URL}/cmdrunner/$CMDRUNNER_VERSION/cmdrunner-$CMDRUNNER_VERSION.jar -O $JMETER_HOME/lib/cmdrunner-$CMDRUNNER_VERSION.jar \
 && java -cp $JMETER_HOME/lib/ext/jmeter-plugins-manager-${JMETER_PLUGINS_MANAGER_VERSION}.jar org.jmeterplugins.repository.PluginManagerCMDInstaller \
 && cd ${JMETER_HOME}/bin && ./PluginsManagerCMD.sh install jpgc-mergeresults \
 && cd ${JMETER_HOME}/bin && ./PluginsManagerCMD.sh install jpgc-synthesis \
 && cd ${JMETER_HOME}/bin && ./PluginsManagerCMD.sh install jpgc-cmd \
 && cd ${JMETER_HOME}/bin && ./PluginsManagerCMD.sh install jpgc-casutg \
 && cd ${JMETER_HOME}/bin && ./PluginsManagerCMD.sh install jpgc-json \
 && cd ${JMETER_HOME}/bin && ./PluginsManagerCMD.sh install jpgc-graphs-additional \
 && cd ${JMETER_HOME}/bin && ./PluginsManagerCMD.sh status \
...
```

**Dockerfile**
- [docker-jmeter-s390x/Dockerfile](https://github.com/cage1016/docker-jmeter-s390x/blob/master/Dockerfile)
- [docker-jmeter-s390x/Dockerfile.s390x](https://github.com/cage1016/docker-jmeter-s390x/blob/master/Dockerfile.s390x)

**Container images**

- [ghcr.io/cage1016/jmeter:5.4.1](https://github.com/cage1016/docker-jmeter-s390x/pkgs/container/jmeter)
- [ghcr.io/cage1016/jmeter-s390x:5.4.1](https://github.com/cage1016/docker-jmeter-s390x/pkgs/container/jmeter-s390x)

## Jmeter Test Plan

在準備好了靜態網站及 Docker 版本的 Jmeter 之後，接下來就是決定 Jmeter Test Plan，由於最終測試會有硬體加速支援的部份，情境就是針對 Nginx + Nginx 啟用 gzip 功能來對比，因此 Test Plan 就會是以 HTTP Request 為主

{{<image src="./img/a.jpg" alt="Simple HTTP Request">}}

Jmeter 中也提供了 Test Script Recorder 的工具，可以搭配 Jmeter Proxy + Firefox 一起使用，這樣就可以省去手動建立那些 HTTP Request 繁鎖的動作，非常的方便

{{<image src="./img/placeholder.jpg" alt="Complex ">}}

- [docker-jmeter-s390x/ap.jmx](https://github.com/cage1016/docker-jmeter-s390x/blob/master/ap.jmx)

## Usage

在準備好 Test Plan 之後就可以透過 Docker 版本的 Jmeter 直接執行

1. Pull Docker image for x86 or x390x
    ```bash
    # x86
    $ docker pull ghcr.io/cage1016/nginx-website-gz:0.1.0
    $ docker pull ghcr.io/cage1016/nginx-website:0.1.0
    $ docker pull ghcr.io/cage1016/jmeter:5.4.1
    
    # s390x
    $ docker pull ghcr.io/cage1016/nginx-website-s390x:0.1.0
    $ docker pull ghcr.io/cage1016/nginx-website-gz-s390x:0.1.0
    $ docker pull ghcr.io/cage1016/jmeter-s390x:5.4.1
    ```

1. Dowonload `jmeter.sh` and test jmx `ap.jmx`

    ```bash
    $ wget https://raw.githubusercontent.com/cage1016/docker-jmeter-s390x/master/jmeter.sh && chmod +x jmeter.sh
    $ wget https://raw.githubusercontent.com/cage1016/docker-jmeter-s390x/master/ap.jmx
    ```

1. Start Nginx
    ```bash
    # x86
    $ docker run --rm -d -p 8080:80 ghcr.io/cage1016/nginx-website-gz:0.1.0

    # x390s
    $ docker run --rm -d -p 8080:80 ghcr.io/cage1016/nginx-website-gz-x390s:0.1.0
    ```

1. Run Jmeter with Docker
    ```sh
    $ ./jmeter.sh
    Error: Please specify JMX using -f.
    Usage: jmeter.sh [-d <deamon>] [-i <jmeter_docker_image>] [-f <jmx_file>] [-t <test_folder>] [-z <enable_tar_html>]     [-l <jmeterVariablesList>]
     -d : Deamon, docker/podman (default: docker)
     -t : Test directory (default: ./tmp)
     -i : Jmeter docker image
     -f : Specify JMX file
     -l : Specify env list of Jmeter in following format: prop01=XXX,bbb=YYY,ccc=ZZZ
     -z : Enable tar html report (default: false)

      Example1: jmeter.sh -f ap.jmx
      Example2: jmeter.sh -i ghcr.io/cage1016/jmeter:5.4.1 -f ap.jmx
      Example3: jmeter.sh -i ghcr.io/cage1016/jmeter:5.4.1 -f ap.jmx -l prop01=XXX,prop02=YYY
    ```
1. Run test `ap.jmx`
    ```bash
    $ ./jmeter.sh -i ghcr.io/cage1016/jmeter:5.4.1 -f ap.jmx -t ap -z true -l TARGET_HOST=localhost,TARGET_PORT=8080,THREADS=1,RAMD_UP=1,DURATION=10
    
    docker run --rm --name jmeter --network host -i -v ${PWD}:${PWD} -w ${PWD} ghcr.io/cage1016/jmeter:5.4.1 ap.jmx -l ap/jmeter.jtl -j ap/jmeter.log   -JTARGET_HOST=localhost -JTARGET_PORT=8080 -JTHREADS=1 -JRAMD_UP=1 -JDURATION=10 -o ap/report -e
    
    Creating summariser <summary>
    Created the tree successfully using ap.jmx
    Starting standalone test @ Wed Dec 29 08:39:40 GMT 2021 (1640767180391)
    Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445
    Warning: Nashorn engine is planned to be removed from a future JDK release
    summary =   8157 in 00:00:10 =  812.8/s Avg:     1 Min:     0 Max:    36 Err:     0 (0.00%)
    Tidying up ...    @ Wed Dec 29 08:39:50 GMT 2021 (1640767190744)
    ... end of run
    
    ==== jmeter.log ====
    See jmeter log in ap/jmeter.log
    ==== Raw Test Report ====
    See Raw test report in ap/ap.jmx.jtl
    ==== HTML Test Report ====
    See HTML test report in ap/report/index.html
    ==== Tar report ====
    See Tar file in ap/1640767193.tar.gz
    ```

1. 檢視產出
   - ./ap/jmeter.log: Jmeter 執行的 Log 檔案
   - ./ap/jmeter.jtl: Jmeter UI Linstern 載入資料
   - ./ap/report/index.html: HTLM 報表

## Jmeter Youtube toturial

{{< youtube SoW2pBak1_Q >}}
