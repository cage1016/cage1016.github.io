# weddingcnp via GCP - 1. 專案架構切分


<!--more-->

此篇就對 [Cage & Ping wedding](http://weddingcnp.appspot.com/) 中實作的專案架構進行簡單的說明，每一個 Google App Engine Service 實作的細節會在後序的篇幅中介紹

#### weddingcnp 系例傳送門

1. [weddingcnp via GCP 簡介](https://kaichu.io/2017/06/08/weddingcnp-via-gcp/)
1. [weddingcnp via GCP - 1. 專案架構切分](https://kaichu.io/2017/06/12/weddingcnp-via-gcp-1/)
1. [weddingcnp via GCP - 2. 前端頁面設計實作](https://kaichu.io/2017/06/18/weddingcnp-via-gcp-2/)
1. [weddingcnp via GCP - 3. endpointAPI 設計實作](https://kaichu.io/2017/07/12/weddingcnp-via-gcp-3/)
1. weddingcnp 前端 vue.js 設計實作
1. weddingcnp edm 寄送 sendgrid

#### weddingcnp 專案架構

[Cage & Ping wedding](http://weddingcnp.appspot.com/) 完全架構在 Google App Engine 上，再功能任務上需要達到幾個要求

- 服務靜態頁面, 一堆的 html/css/javascript 及些許的動態資料
- 作為 API 的伺服器，收集前端表單送過來的資料並儲存至 Datastore 中
- webmasters domain owner

__project structure__

```bash
.
├── endpoints               // endpoints instance
│   ├── apis
│   ├── handler
│   ├── lib
│   ├── app.py
│   ├── app.pyc
│   ├── app.yaml
│   └── ...
├── frontend                // frontend instance
│   ├── public              // 公開資源
│   │   ├── audio           // 音樂檔
│   │   ├── css             // 靜態網頁用 css
│   │   ├── images          // 靜態網頁用 image
│   │   └── js              // 靜態網頁用 javascript
│   ├── templates           // golang 模版
│   │   ├── bingo.tmpl
│   │   ├── .....
│   │   └── rsvp.tmpl
│   ├── app-engine.go
│   ├── app-standalone.go
│   ├── app.go
│   ├── app.yaml
│   ├── index.yaml
│   └── main.go
├── ownership               // webmaster instance
│   ├── app.yaml
│   └── googleae8f4bcce8bec00c.html
├── dispatch.yaml           // dispatch.yaml
└── makefile                // cli helper makefile
```

整個專案的檔案架構如上，可以分為 `endpoints`、`frontend`、`ownership` 三個 Service(Module), 而 Google App Engine 可以非常有彈性的透過 Service[^1] 的方式針對不同的任務給予不同資源[^2]，而 [Cage & Ping wedding](http://weddingcnp.appspot.com/) 基本上使用預計的部份就足夠了，不需特別指派

__dispatch.yaml__

```yaml
dispatch:
  - url: "*/_ah/spi/*"
    module: default

  - url: "*/tasks/*"
    module: default

  - url: "*/favicon.ico"
    module: frontend

  - url: "*/googleae8f4bcce8bec00c.html"
    module: ownership

  - url: "*/*"
    module: frontend
```

上述的 `dispatch.yaml` 可以幫我們重導流量至我們設定的 Service(Module)，不過在部署的部份則是需要一個一個 Service(Module)來進行部署，部署完之後會在 Google App Engine Services 中看到所有的 Services

{{<image src="img/weddingcnp-via-gcp-1_1.png">}}

再則我們可以透過 version[^3] 的方式來管理部署的 Service

{{<image src="img/weddingcnp-via-gcp-1_2.png">}}

當 Google App Engine Service 切分的越細，雖然可以任務的資源給予更細的調整，不過卻也造成部署上的複雜度，因此可以透過 `mikefile` 來加速部署的流程與速度

__makefile__

```bash
ACCOUNT = cage.chung@gmail.com
PROJECT = weddingcnp
VERSION = b7484dd

NODE_BIN = $(shell npm bin)

set_config:
	gcloud config set account $(ACCOUNT)
	gcloud config set project $(PROJECT)

install:
	# endpoints dependency pakcage
	pip install -r endpoints/requirements.txt -t endpoints/lib

run:
	# node devServer.js &
	#goapp serve	dispatch.yaml frontend/app.yaml	endpoints/app.yaml ownership/app.yaml
	dev_appserver.py dispatch.yaml frontend/app.yaml endpoints/app.yaml ownership/app.yaml --appidentity_email_address=save-avatar@weddingcnp.iam.gserviceaccount.com --appidentity_private_key_path=/Users/cage/.gvm/pkgsets/go1.7/global/src/github.com/cage1016/weddingcnp/endpoints/weddingcnp-3a652c96507d.pem --skip_sdk_update_check=yes --host 0.0.0.0 --enable_sendmail=yes

update_frontend:
	goapp deploy -application $(PROJECT) -version $(VERSION) frontend/app.yaml

update_endpoints:
	goapp deploy -application $(PROJECT) -version $(VERSION) endpoints/app.yaml

update_ownership:
	appcfg.py update --application=$(PROJECT) --version=1 ownership/app.yaml

update_dispatch:
	appcfg.py --application=$(PROJECT) --version=$(VERSION) update_dispatch .

update_index:
	appcfg.py --application=$(PROJECT) --version=$(VERSION) update_indexes endpoints

update_queue:
	appcfg.py --application=$(PROJECT) --version=$(VERSION) update_queues endpoints

update: update_frontend update_endpoints update_ownership update_dispatch update_index update_queue
```

[^1]: Service == Module, 以前的名子叫作 Module, 後來改名為 Service，透過 `dispatch.yaml` 來指定，可以針對 Service 給予不同的 runtime、機器資源或是用另一種程式語言混搭
[^2]: [waldogcp](https://www.slideshare.net/cagechung/waldogcp) 一個也是基於 GCP 上建構出來的最佳路徑服務，亦使用 `dispatch.yaml` 來指派，對於服務中運算最重的部份(基因演算法)給予較大的機器
[^3]: 將版號命名為 git 的 commit, 可以讓自己比較容易進行 Service 的版本管控

