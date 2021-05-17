# weddingcnp via GCP - 3. endpointAPI 設計實作


<!--more-->

此篇就對 [Cage & Ping wedding](http://weddingcnp.appspot.com/) 中實作專案中最為重要的 backend API (endpoint API) 部份進行簡單的說明，每一個 Google App Engine Service 實作的細節會在後序的篇幅中介紹

#### weddingcnp 系例傳送門

1. [weddingcnp via GCP 簡介](https://kaichu.io/2017/06/08/weddingcnp-via-gcp/)
1. [weddingcnp via GCP - 1. 專案架構切分](https://kaichu.io/2017/06/12/weddingcnp-via-gcp-1/)
1. [weddingcnp via GCP - 2. 前端頁面設計實作](https://kaichu.io/2017/06/18/weddingcnp-via-gcp-2/)
1. [weddingcnp via GCP - 3. endpointAPI 設計實作](https://kaichu.io/2017/07/12/weddingcnp-via-gcp-3/)
1. weddingcnp 前端 vue.js 設計實作
1. weddingcnp edm 寄送 sendgrid

#### endpointAPI 設計實作

{{<image src="img/weddingcnp-via-gpc-3_1.png">}}

{{<image src="img/weddingcnp-via-gpc-3_2.png">}}

[Cage & Ping wedding](http://weddingcnp.appspot.com/) 的網站大至上可以區分為靜態頁面(frontend Service, Golang) 及負責資料收集的 API (endpoints Service, Python). Google [Endpoints API](https://cloud.google.com/endpoints/docs/frameworks/legacy/v1/python/create_api)[^1] 是架構在 GAE 上的一個 RPC (remote procedure call) 服務。由於 endpoints API 是基於 [ProtoRPC remote.Service](https://cloud.google.com/appengine/docs/standard/python/tools/protorpc/) 實作出來的，所以在實作我們的方法時也是依照 protorpc 的方式來定義，小弟在寫 [Cage & Ping wedding](http://weddingcnp.appspot.com/) 這一個網站的時候 Endpoints API 還是 1.0，所以這篇文章還是以 Endpoints API 1.0 來說明

### endpoint API demo project

小弟我建立一個比較簡單的範例來說明

https://github.com/cage1016/endpoint-api-demo

```bash
# git clone repo
$ git clone git@github.com:cage1016/endpoint-api-demo.git

# install GAE python requirement packages
$ pip install -r requirements.txt -t lib
Collecting arrow==0.8.0 (from -r requirements.txt (line 1))
Collecting python-dateutil (from arrow==0.8.0->-r requirements.txt (line 1))
  Downloading python_dateutil-2.6.1-py2.py3-none-any.whl (194kB)
    100% |████████████████████████████████| 194kB 1.2MB/s
Collecting six>=1.5 (from python-dateutil->arrow==0.8.0->-r requirements.txt (line 1))
  Using cached six-1.10.0-py2.py3-none-any.whl
Installing collected packages: six, python-dateutil, arrow
Successfully installed arrow python-dateutil-2.2 six-1.10.0
...

# run GAE python locally
$ dev_appserver.py app.yaml
INFO     2017-07-12 13:30:19,063 devappserver2.py:116] Skipping SDK update check.
INFO     2017-07-12 13:30:19,706 api_server.py:312] Starting API server at: http://localhost:51100
INFO     2017-07-12 13:30:19,710 dispatcher.py:226] Starting module "default" running at: http://localhost:8080
INFO     2017-07-12 13:30:19,711 admin_server.py:116] Starting admin server at: http://localhost:8000

# visit http://localhost:8080
```

完全上述步驟後透過瀏覽器訪問 `http://localhost:8080/_ah/api/explorer` 就可以得到圖一中類似 [Google API explorer](https://developers.google.com/apis-explorer/#p/) 風格的本地測試頁面, 這是一個非常方便的工具，可以讓自己在本地透過瀏覽器直接操作自己定義的 endpointAPI

{{<image src="img/weddingcnp-via-gpc-3_3.png">}}

#### 專案架構

```
.
├── apis                   // endpoint api packages
│   ├── __init__.py
│   └── books.py
├── lib
├── app.py                 // 主程式進入點
├── app.yaml               // GAE 配置檔
├── appengine_config.py    // lib 載入配置
├── models.py              // GAE Datastore 檔
├── requirements.txt       // GAE python 引用資源庫
├── settings.py            // 專案設定檔
└── utils.py               // helpers function
```

一個 GAE endpoint API Service 大至上進行簡單的配置可以執行了

__app.yaml__

```yaml
module: default

runtime: python27
api_version: 1
threadsafe: yes

handlers:
- url: /_ah/spi/.*
  script: app.API
  secure: always

libraries:
- name: endpoints
  version: latest
```

**app.yaml** 配置上的重點為在 handlers 區塊加入 `/_ah/spi/.*` 這一個 route 並將 route 導至 endpoints API server


__app.py__

```python
import endpoints

from apis import books

API = endpoints.api_server([
    books.BooksAPI
])
```

endpoints api server 由 `endpoints.api_server([])` 建立, 其中的參數是實作 `protorpc.remote.Service` 類別的 API, 目前我們只有實作一個 `books.BooksAPI` 所以只有填入一個

__settings.py__

```python
...

books_api = endpoints.api(name='dummy',
                         version='v1',
                         description='dummy API',
                         allowed_client_ids=[CLIENT_ID,
                                             endpoints.API_EXPLORER_CLIENT_ID],
                         scopes=[endpoints.EMAIL_SCOPE])
```

在我們定義的 `protorpc.remote.Service` API 上加一個 Decorate[^2] 來描述我們的 API(API 名稱、版號、說明、允許呼叫的 GCP CLIENT_ID, scopes 等等)

> @util.positional(2)
def api(name, version, description=None, hostname=None, audiences=None,
        scopes=None, allowed_client_ids=None, canonical_name=None,
        auth=None, owner_domain=None, owner_name=None, package_path=None,
        frontend_limits=None, title=None, documentation=None, auth_level=None):

__api/books.py__

```python
@books_api.api_class(resource_name="books")
class BooksAPI(remote.Service):
  """
  Books API
  """

  @endpoints.method(BOOKS_LIST_RESOURCE, BookForms, path="books",
                    http_method="GET", name="list")
  def listBooks(self, request):
      """List books"""
      ...

  @endpoints.method(BOOKS_CREATE_RESOURCE, BookForm, path="books",
                    http_method="POST", name="post")
  def addBook(self, request):
      """Add books"""
      ...

  @endpoints.method(BOOKS_DELETE_RESOURCE, BooleanMessage, path="books/{websafeKey}",
                    http_method="DELETE", name="delete")
  def deleteBook(self, request):
      """Delete book"""
      ...

  @endpoints.method(BOOKS_UPDATE_RESOURCE, BookForm, path="books/{websafeKey}",
                    http_method="PUT", name="put")
  def updateBook(self, request):
      """Update book"""
      ...
```

在定義 resource 下的方法時依然使用 Decorate 的方式來對方法進行設置 (method name, path, http method,
  cache control, scopes, audiences, client ids and auth_level 等)

> @util.positional(2)
def method(request_message=message_types.VoidMessage,
           response_message=message_types.VoidMessage,
           name=None,
           path=None,
           http_method='POST',
           cache_control=None,
           scopes=None,
           audiences=None,
           allowed_client_ids=None,
           auth_level=None):


設置完 endpointAPI 的基本架構，剩下的部份就是 GAE Python[^3] 的基本操作，定義 Datastore module、對 Datastore 進行 CRUD 的操作。有興趣的朋友可以把專案 clone 下來玩看看，並試著自己俢改修改會更有感覺。

endpointAPI 其他的功能還有很多，可以搭配 gRPC[^4]一起使用、建立 Android/iOS 的 frameworks[^5] 等，詳細的說明可以見官方的文件 [How-to Guides](https://cloud.google.com/endpoints/docs/how-to)

### curl

透過瀏覽器操作過後，透過 curl 操作的結果也是一樣的

```
$ curl http://localhost:8080/_ah/api/dummy/v1/books
{
 "items": [
  {
   "name": "Shel Silverstein",
   "title": "The Giving Tree",
   "websafeKey": "aghkZXZ-Tm9uZXIZCxIEQm9vayIPVGhlIEdpdmluZyBUcmVlDA"
  },
  {
   "name": "Harper Lee",
   "title": "To Kill a Mockingbird",
   "websafeKey": "aghkZXZ-Tm9uZXIfCxIEQm9vayIVVG8gS2lsbCBhIE1vY2tpbmdiaXJkDA"
  },
  {
   "name": "dfad",
   "title": "fdafd",
   "websafeKey": "aghkZXZ-Tm9uZXIPCxIEQm9vayIFZmRhZmQM"
  }
 ]
}
```

[^1]: Endpoints API 2.0 的版本已經出來了，相較於 1.0 改變滿大的，原先像 Google API explorer 的部份由 swagger 取代，詳細的說明請見官網
[^2]: 在定義 API 說明時有二種方式。這邊採用的是 `@books_api.api_class`，在 `books_api` 下允許定義多組 RESTful resource
[^3]: [go-endpoints](https://github.com/GoogleCloudPlatform/go-endpoints) 其實 go 版的寫法更為簡潔一點
[^4]: [gRPC API Service](https://cloud.google.com/endpoints/docs/grpc-service-config)
[^5]: [Cloud Endpoints Frameworks for App Engine](https://cloud.google.com/endpoints/docs/frameworks/java/about-cloud-endpoints-frameworks)

