# Managed VMs lab


<!--more-->

**Presentation**:

- slide: [GAE Managed VM Introduction - Google Slides](https://goo.gl/SwuEvR)
- source code: [cage1016/managed-vms-lab](https://github.com/cage1016/managed-vms-lab)

## GAE Overview

[Google App Engine](https://cloud.google.com/appengine/docs) 是 [Google Cloud Platform](https://cloud.google.com/) 中一個 PasS (Platform as a service) 的服務. 在 Pass 的 GAE 中，開發者只需要專注於 **Application** 及 **Data**，
其於的 **Runtime**、**Middleware**、**OS**、**Virtualization**、**Servers**、**Storage**、**Networking** 則完全被 Google 控制管理。

_GAE Architecture_
{{<image src="img/01_WebApp_ArchDiagram.png">}}
picture: https://cloud.google.com/solutions/architecture/webapp

App Engine 目前提供的執行環境有 `Python`、`Golang`、`Java`及`PHP` 4 種程式語言。支援多版本應用程式實例(multiple version)，
且可以直接在 Console 介面中進行版本的切換，也支援流量分流來方便進行 A/B 測試。

App Engine 也整合了 Memcache 及 Tasks Queue 的服務。Memcache 是一種記憶體快取並可以共享在 App Engine 的實例中，
它可以有效的提高資料存取的速度(如帳號資料從 Datastore 讀取後放至 Memcache 一份)。

Tasks Queues 則提供了一種處理長時間請求的方式(600s)。而 App Engine 也內建了 load balancer。

### Docker Overview

Docker 是一種輕量的 container 技術，不同於 VM (Virtual Machine) 的地方是 Container 是在作業系統層面上實作虛擬化，直接使用本地主機的作業系統，
而傳統方式則是在硬體層面實作。


{{<image src="img/virtualization.png" caption="Virtualization">}}
vs.
{{<image src="img/docker.png" caption="Docker">}}
picture: https://philipzheng.gitbooks.io/docker_practice/content/introduction/what.html

**Why Docker?**

> 作為一種新興的虛擬化方式，Docker 跟傳統的虛擬化方式相比具有眾多的優勢。

> 首先，Docker 容器的啟動可以在秒級實作，這相比傳統的虛擬機方式要快得多。 其次，Docker 對系統資源的使用率很高，一台主機上可以同時執行數千個 Docker 容器。

> 容器除了執行其中應用外，基本不消耗額外的系統資源，使得應用的效能很高，同時系統資源消耗更少。傳統虛擬機方式執行 10 個不同的應用就要啟動 10 個虛擬機，而 Docker 只需要啟動 10 個隔離的應用即可。

> 具體說來，Docker 在以下幾個方面具有較大的優勢。

> **更快速的交付和部署**

> 對開發和維運（DevOps）人員來說，最希望的就是一次建立或設定，可以在任意地方正常執行。

> 開發者可以使用一個標準的映像檔來建立一套開發容器，開發完成之後，維運人員可以直接使用這個容器來部署程式碼。 Docker 可以快速建立容器，快速迭代應用程式，並讓整個過程全程可見，使團隊中的其他成員更容易理解應用程式是如何建立和工作的。 Docker 容器很輕很快！容器的啟動時間是秒級的，大量地節約開發、測試、部署的時間。

> **更有效率的虛擬化**

> Docker 容器的執行不需要額外的虛擬化支援，它是核心層級的虛擬化，因此可以實作更高的效能和效率。

> **更輕鬆的遷移和擴展**

> Docker 容器幾乎可以在任意的平台上執行，包括實體機器、虛擬機、公有雲、私有雲、個人電腦、伺服器等。 這種兼容性可以讓使用者把一個應用程式從一個平台直接遷移到另外一個。

> **更簡單的管理**

> 使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分發和更新，從而實作自動化並且有效率的管理。

> 擷錄於 [為什麼要用 Docker | 《Docker —— 從入門到實踐 》正體中文版](https://philipzheng.gitbooks.io/docker_practice/content/introduction/why.html)

## Managed VMs

> App Engine + Docker = Managed VMs

App Engine 是 Google 一個 PasS 的服務，完全受 Google 管理及控制，開發者只需要專注於 **Data** 及 **Application** 的發開。

Docker 則是一個新興的 container 技術，讓發開者可以更快速的交付和部署、更有效率的虛擬化、更輕鬆的遷移和擴展、更簡單的管理。

**Why Managed VMs?**

開發 App Engine，開發者雖然只要專注於 **Data** 及 **Application** 即可，但是也因為為享有諸多的服務而有所限制(目前只有支援 4 種程式語言 Golang, Python, Java, PHP、在 sandbox 中無法寫入檔案等限制)
，而 Managed VMs 中引進了 docker 的元素讓 App Engine 有了更多的彈性。如: **可以進行檔案讀寫**、**自定 Runtime**。

{{<image src="img/Managed-VMs.jpg">}}

在使用 Managed VMs 時與一般的 App Engine 應用程式無異，差異的部份是在 `app.yaml` 中多設置了 `vm:true` 的配置

_golang, app.yaml_

```shell
# make sure to replace "projectid" below with the project ID configured in the Google Developer Console
runtime: go
api_version: go1
# The vm setting is what determines if we are using a sandbox or MVM
vm: true
module: module1

automatic_scaling:
  min_num_instances: 1
  max_num_instances: 5
  cool_down_period_sec: 60
  cpu_utilization:
    target_utilization: 0.2

resources:
  cpu: .5
  memory_gb: 1.3

handlers:
- url: /.*
  script: _go_app
```
上述 `app.yaml` 除了啟用了 Managed VMs 也配置了自動擴展配置及給予特定的資源(cpu, memory)

## Demo

source code: [cage1016/managed-vms-lab](https://github.com/cage1016/managed-vms-lab)

_Demo Architecture_

{{<image src="img/demo-managed-vms-architecture.jpg">}}

_dispatch.yaml_

```yaml
dispatch:
  - url: "*/favicon.ico"
    module: default

  # appengine sandbox
  - url: "your-project-id.appspot.com/"
    module: default

  # managed vms
  - url: "*/module1/*"
    module: module1

  - url: "*/*"
    module: default
```

本次 demo lab 的架構為一個 sandbox runtime(default module) + managed VMs(module1) with automatic scaling. Route request 可以透過 `dispatch.yaml` 的設定進行重導[1]

_golang, file structure (same as python)_

```shell
├── create_ab_test_instances.sh     # ab test instances create shell
├── default                         # default module
│   ├── app.yaml                    # default moduel app.yaml
│   ├── dispatch.yaml               # dispatch.yaml
│   └── sandbox.go                  # default module web app code
├── local_run.sh                    # local run shell
├── module1                         # module 1 module
│   ├── Dockerfile                  # auto-gen by gcloud command
│   ├── app.yaml                    # module1 module app.yaml
│   └── module1.go                  # module1 module web app code
├── update_all.sh                   # app upload shell (default, module1)
```

## Run locally

> 本次 Demo 使用 Google Cloud SDK 0.9.79

```shell
# install Google cloud SDK
$ curl https://sdk.cloud.google.com | bash

# update components
$ gcloud components update

# clone repo
$ git clone https://github.com/cage1016/managed-vms-lab
```

重新命名 \*.yaml.exist --> \*.yaml

  - \*/default/`dispatch.yaml.exist` --> \*/default/`dispatch.yaml`
  - \*/default/`update_all.sh.exist` --> \*/defalt/`update_all.sh`
  - \*default/`create_ab_test_instances.sh.exist` --> \*/default/`create_ab_test_instances.sh`

```shell
# run locally
$ cd golang (or cd python)

# execute run bash
$ sh local_run.sh
```

## Deploy to GAE

在發佈 demo 應用程式至 App Engine 時，需要先配置一些參數. Golang/Python 的配置方式都一樣。

1. 在發佈應用程式至 App Engine 之前，確認你有先在 [Google Developers Console](https://goo.gl/JkWVb9) 建立專案
2. 設置專案 `PROJECT_ID`
  - \*/default/dispatch.yaml
  - \*/default/update_all.sh
  - \*/create_ab_test_instances.sh
3. `$ sh update_all.sh` 就會依序發佈 App Engine sandbox 及 Managed VMs 至 App Engine
4. 訪問
  - default module: https://your-project-id.appspot.com/
  - module1: https://your-project-id.appspot.com/module1/
  - module1 route: https://your-project-id.appspot.com/module1/sayhi


## Apache ab test

在 module1 的 Managed VMs 中有設定 automatic scaling, 且在 cpu > **0.2** 下會自動進行 scaling

```shell
# cd folder
$ cd golang (or cd python)

# create test instances
$ sh create_ab_test_instances.sh

# ssh 至 gce instances
$ gcloud compute ssh your-project-id --zone=project-zone-name

# gce instances setup
$ sudo yum -y upgrade

# install Apache ab tset
$ yum provides /usr/bin/ab
$ sudo yum install httpd-tools

# test
$ ab -n 10000 http://your-project-id.appspot.com/module1/
```

在 Apache ab test 壓測 module1 的模組，就可以透過 `$gcloud compute instances list --project=your-project-id` 會自動 scale out

## Reference
1. App Engine Module:
  - [App Engine Modules in Python](https://goo.gl/p3nl48)
  - [App Engine Modules in Go](https://goo.gl/KpdKpF)

