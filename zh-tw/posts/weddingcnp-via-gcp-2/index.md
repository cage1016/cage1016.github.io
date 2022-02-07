# weddingcnp via GCP - 2. 前端頁面設計實作


<!--more-->

此篇就對 [Cage & Ping wedding](http://weddingcnp.appspot.com/) frontend Service 使用了 GAE standard runtime 搭配 echo 網頁框架實作介紹

#### weddingcnp 系例傳送門

1. [weddingcnp via GCP 簡介](https://kaichu.io/2017/06/08/weddingcnp-via-gcp/)
1. [weddingcnp via GCP - 1. 專案架構切分](https://kaichu.io/2017/06/12/weddingcnp-via-gcp-1/)
1. [weddingcnp via GCP - 2. 前端頁面設計實作](https://kaichu.io/2017/06/18/weddingcnp-via-gcp-2/)
1. [weddingcnp via GCP - 3. endpointAPI 設計實作](https://kaichu.io/2017/07/12/weddingcnp-via-gcp-3/)
1. weddingcnp 前端 vue.js 設計實作
1. weddingcnp edm 寄送 sendgrid

#### weddingcnp 前端頁面設計實作

[Cage & Ping wedding](http://weddingcnp.appspot.com/) **frontend Service(Module)** 主要的功能如下

- serve 靜態資源(audio、js、images、css)，serve 動態資源(婚紗照片由 Google Cloud Storage 提供)
- 服務整個主體的網站架構中模版
  - [首頁介紹頁](https://weddingcnp.appspot.com/)
  - [婚妙相簿頁](https://weddingcnp.appspot.com/gallery)
  - 婚宴地點介紹引導頁([訂婚](https://weddingcnp.appspot.com/luzhu)、[結婚](https://weddingcnp.appspot.com/chiayi))
  - [報名頁面](https://weddingcnp.appspot.com/rsvp)(我要參加)[^1]
  - [隱藏網頁](https://weddingcnp.appspot.com/greeting)(求婚影片搶先看)，連結由喜帖中的 QR code 提供
  - [bingo 遊戲](https://weddingcnp.appspot.com/bingo)

{{<image src="img/weddingcnp-via-gpc-2_1.png">}}

**frontend Service**

```bash
.
├── public                // static resource
│   ├── audio
│   ├── css
│   ├── images
│   └── js
├── templates             // html templates
│   ├── bingo.tmpl        // bingo game
│   ├── chiayi.tmpl       // 地點引導
│   ├── gallery.tmpl      // 婚紗相簿
│   ├── greeting.tmpl     // 隱藏網頁，連結由喜帖中的 QR code 提供
│   ├── index.tmpl
│   ├── layout.tmpl
│   ├── live.tmpl
│   ├── luzhu.tmpl        // 地點引導
│   └── rsvp.tmpl         // 報名頁面
├── app-engine.go
├── app-standalone.go     // Google App Engine entry point
├── app.go
├── app.yaml              // frontend services yaml
├── index.yaml            // index for Datastore
└── main.go               // echo framework entry point
```

## Google App Engine Basic

[Google App Engine](https://cloud.google.com/appengine/docs) 現在有提供 **Standard** 及 **Flexible** 的版本

[Standard](https://cloud.google.com/appengine/docs/standard/):
- Python 2.7
- Java 7
- PHP 5.5
- Go 1.6[^3]

[Flexible](https://cloud.google.com/appengine/docs/flexible/):
- Python 2.7/3.5
- Java 8
- Node.js
- Go 1.8
- Ruby
- PHP 5,6,7
- .Net
- Custom Runtimes

二者最大的差異是 **Standard** runtime 為 Google Fully management，所以享有每天 **28** instance FREE 的配額[^4]，**Flexible** runtime 是以 GCE 的 instance 來跑，所以基本上的計價方式就是以 GCE 的計算方式，以分計費(最少收 10 分鐘的錢)，且因為是 GCE 的方式來跑，因此彈性上就會比 **Standard** runtime 的來的大，不過缺點就是一開機器就開始算錢惹。在 weddingcnp 專案一開始就希望能夠少花一點錢，評估上 **Standard** runtime 就已經滿足需求

[Cage & Ping wedding](http://weddingcnp.appspot.com/) **frontend Service(Module)** 選擇使用了 **Standard** runtime 中 `Go` 語言的實作方式。由於前端需求並不複雜且 `Go` 實作的方式要比 `Python` 來的簡潔，部署至 Google App Engine 上的專案大小、記憶體的表現都上 `Go` 的表現要來的比 `Python` 來的要好，因此選擇了 `Go` 來實作。

> 各位朋友就端看自己的需求來評估要使用那一種 runtime

## Google App Engine: hello example

__app.yaml__

```yaml
runtime: go
api_version: go1

handlers:
- url: /.*
  script: _go_app
```

__hello.go__

```go
package hello

import (
    "fmt"
    "net/http"
)

func init() {
    http.HandleFunc("/", handler)
}

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello, world!")
}
```

``` bash
# start GAE server
$ dev_appserver.py app.yaml
INFO     2017-06-18 04:02:36,963 devappserver2.py:692] Skipping SDK update check.
INFO     2017-06-18 04:02:37,519 api_server.py:272] Starting API server at: http://localhost:54584
INFO     2017-06-18 04:02:37,522 dispatcher.py:205] Starting module "default" running at: http://localhost:8080
INFO     2017-06-18 04:02:37,524 admin_server.py:116] Starting admin server at: http://localhost:8000

# fetch
$ curl http://localhost:8080
Hello, world!
```

## Weddingcnp frontend

`Go` 由語言中原生就有提供 `net/http` 的 package 可以使用，所以可以很輕易的建立一個 Google App Engine `Go` **Standard** 的應用程式，不過 Google App Engine 還是可以搭配使用其他的網頁框架: [gin-gonic/gin](https://github.com/gin-gonic/gin)、[echo](https://github.com/labstack/echo) 等，weddingcnp 使用了 `echo` 這一套輕量的網頁框架

Google App Engine **Standard** 是由 Google fully management，因此在使用 echo 框架[^5]需要特別的的調整一下


__app-engin.go__

```go
// +build appengine

package main

import (
    "github.com/labstack/echo"
    "github.com/labstack/echo/engine/standard"
    "net/http"
)

func createMux() *echo.Echo {
    e := echo.New()

    // note: we don't need to provide the middleware or static handlers, that's taken care of by the platform
    // app engine has it's own "main" wrapper - we just need to hook echo into the default handler
    s := standard.New("")
    s.SetHandler(e)
    http.Handle("/", s)

    return e
}
```

__app-standalone.go__

```go
// +build !appengine,!appenginevm

package main

import (
	"github.com/labstack/echo"
	"github.com/labstack/echo/engine/standard"
	"github.com/labstack/echo/middleware"
)

func createMux() *echo.Echo {
	e := echo.New()

	e.Use(middleware.Recover())
	e.Use(middleware.Logger())
	e.Use(middleware.Gzip())

	e.Use(middleware.Static("public"))

	return e
}

func main() {
	e.Run(standard.New(":8080"))
}
```

__app.go__

```go
package main

// reference our echo instance and create it early
var e = createMux()
```

__main.go__

```go
package main

...

/**
 * RSVP page
 */
func rsvpGET(c echo.Context) error {
	var p = &Page{}
	p.Name = "slider-title-page"
	p.MenuIconWhite = true

	r.HTML(c.Response().(*standard.Response).ResponseWriter, http.StatusOK, "rsvp", p)
	return nil
}

...

func init() {
	e.GET("/", indexGET)
	e.GET("/gallery", galleryGET)
	e.GET("/chiayi", chiayiGET)
	e.GET("/luzhu", luzhuGET)
	e.GET("/rsvp", rsvpGET)
	e.GET("/greeting", greetingGET)
	e.GET("/live", liveGET)
	e.GET("/bingo", bingoGet)
	e.GET("/img", imgServe)

	e.SetHTTPErrorHandler(ErrorHandler)
}
```

透過 `app-engine.go`、`app-standalone.go`、`app.go`、`main.go` 的調整，就可以在 Google App Engine **Standard** runtime 上使用 `echo` 框架(不過僅限 Go 1.6)。在頁面 Render 的部份則是使用 `github.com/unrolled/render` 模版系統來建置。相簿的照片是放在 Google Cloud Storage 透過 Image API 來取存[^2]，大至上都是很單純的頁面

{{<image src="img/weddingcnp-via-gpc-2_2.png">}}

此篇大至上說明 [Cage & Ping wedding](http://weddingcnp.appspot.com/) **frontend Service(Module)** 實作的方式。比較可惜的是目前 `Go` 主要的版本已經到 1.8.3, 1.9 Beta 也公佈了，而 Google App Engine **Standard** runtime 只有支援 `Go` 1.6 (1.8 應該快要支援), 當 Google App Engine **Standard** runtime 支援 `Go` 1.8 時才會有比較多的框架來搭配，目前就使用容易性來說還是 **Flexible** runtime 較好

[^1]: 報名頁面實作細節會再後序的篇幅說明
[^2]: [GAE go Image API for GCS · Kaichu.io](http://localhost:4000/2016/10/31/gae-go-image-api/)
[^3]: Google App Engine **Standard** 支援 1.8 的版本應該快了(1.9 Beta 都出來了 XD)
[^4]: Google App Engine **Standard** 享有一些免費的配額，細項請看 [Quotas](https://cloud.google.com/appengine/quotas)
[^5]: [Google App Engine Recipe | Echo - High performance, minimalist Go web framework](https://echo.labstack.com/cookbook/google-app-engine#standalone), 由於 GAE 目前只有支持 Go 1.6，且 Go 1.7 以後將 `context` 改為內建 package，一些依賴的 package 也跟著升級了，所以目前建議使用 **Flexible** 會比較簡單

