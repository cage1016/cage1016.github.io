# golang serving a single page application


<!--more-->

## golang serve SPA

最近有前端開發的需求，選用了 [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit) 來進行二次開發，省去一些想要使用 React, redux, redux-router 的基本配置，這樣速度會快一點

因為 `react-redux-starter-kit` 也使用 webpack 進行程式碼的打包, 所以最後的產出預設在 `dist` 資料夾中，所以部署時只需要一個簡單的 host server 即可

```bash
# dist example
$ l
total 752
-rw-r--r--  1 cage  staff    3317 Aug  4 00:22 1.counter.fa53ea42bc9ff9de19bd.js
-rw-r--r--  1 cage  staff  144953 Aug  4 00:22 app.5d0f2ab61ef7dd5daac5.js
-rw-r--r--  1 cage  staff    2619 Aug  4 00:22 app.97a1751c9624097874a4b54cb93fa067.css
-rw-r--r--  1 cage  staff     173 Aug  4 00:33 app.go
-rw-r--r--  1 cage  staff   24838 Aug  4 00:22 favicon.ico
-rw-r--r--  1 cage  staff     103 Aug  4 00:22 humans.txt
-rw-r--r--  1 cage  staff     604 Aug  4 00:22 index.html
-rw-r--r--  1 cage  staff      24 Aug  4 00:22 robots.txt
-rw-r--r--  1 cage  staff  183224 Aug  4 00:22 vendor.9012d9d99074521f418e.js
```

考慮效能的問題, 最後打算使用 golang 來當作 host server, golang 內建的 `net/http` 可以輕鬆的使用 `http.FileServer(http.Dir("./"))` 來 host 整個靜態目錄

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	log.Println("Listening port 3000...")
	log.Fatal(http.ListenAndServe(":3000", http.FileServer(http.Dir("./"))))
}
```

但是上述的作法基本上是可以動的, 不過如果前端自己有使用到 `redux-router` 時, golang 並不會將請求導至前端的 router 而是直接得到 golang 404 而不會進到前端 redux-router 訂定的 router (如果有使用 redux-router 對 Notfound 進行處理)

```bash
http://localhost:3000/dfa
```

{{<image src="img/golang-serve-static-site-404-golang.png">}}

所以我們使用了 golang `echo` 的 web framework, 監聽所有的請求並直接導至 `index.html` 的前端靜態檔案

```go
package main

import (
	"flag"
	"fmt"
	"net/http"
	"os"
	"strings"

	"github.com/labstack/echo"
	"github.com/labstack/echo/engine/standard"
	mw "github.com/labstack/echo/middleware"
)

const (
	wwwRoot = "./"
)

var (
	httpPort = flag.Int("http", 3000, "http port number")
)

func Init() *echo.Echo {

	e := echo.New()

	e.Debug()

	e.Use(mw.Logger())
	e.Use(mw.Recover())

	e.Any("/*", echo.HandlerFunc(func(c echo.Context) (err error) {
		r := c.Request().(*standard.Request).Request
		w := c.Response().(*standard.Response).ResponseWriter

		requestPath := r.URL.Path
		fileSystemPath := wwwRoot + r.URL.Path
		endURIPath := strings.Split(requestPath, "/")[len(strings.Split(requestPath, "/"))-1]
		splitPath := strings.Split(endURIPath, ".")
		splitLength := len(splitPath)
		if splitLength > 1 && splitPath[splitLength-1] != "go" {
			f, error := os.Stat(fileSystemPath)
			if error == nil && !f.IsDir() {
				http.ServeFile(w, r, fileSystemPath)
				return
			}
		}
		http.ServeFile(w, r, wwwRoot+"index.html")
		return
	}))

	return e
}

func main() {
	flag.Parse()

	server := Init()
	server.Run(standard.New(fmt.Sprintf(`:%d`, *httpPort)))
}
```

```bash
http://localhost:3000/dfa
```

{{<image src="img/golang-serve-static-site-404.jpg">}}

## run SPA as with docker image

處理完 golang 對 single page application 的支援, 進一步簡化部署的流程, 可以將整個 single page application 連同 `app.go` 利用 golang build 的方式編譯成執行檔, 再將 golang 執行檔透過 `Dockerfile` 編譯成 docker image, 這樣一來就可以很容易的部署在任何可以執行 container 的環境

{{<image src="img/golang-serve-static-site-flow.jpg">}}

上述一系列的流程我們可以使用 `npm run scripts` 的方式把它完全串起來來達到一鍵建立 docker image

```Dockerfile
FROM alpine:3.3

MAINTAINER cage.chung <cage.chung@gmail.com>

WORKDIR /go
ADD . /go/

EXPOSE 3000

CMD ["./counter"]
```

Dockerfile 檔案中我們需要指定 docker image 啟動時直接執行我們透過 `GOOS=linux GOARCH=amd64 go build -o counter` 編譯出來的執行檔 `counter*`

執行自訂義 npm scripts `$ npm run docker` 成功執行後會自動建立 docker image

```bash
# list docker image
$ docker images
REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
username/counter                               v0.1.0              1771ddbe0a98        4 seconds ago       14.67 MB
```

執行 docker image

```bash
# run docker image
$ docker run -d --name counter -p 3000:3000 username/counter:v0.1.0
f8394fec624a4e3b989f7ce48857f64178e39aa8a5195f39e2d0d5a6572ee55c

# docker ps
$ dps
CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS              PORTS                    NAMES
f8394fec624a        username/counter:v0.1.0   "./counter"         2 seconds ago       Up 1 seconds        0.0.0.0:3000->3000/tcp   counter
```

## repo

Demo example [cage1016/golang-serve-spa](https://github.com/cage1016/golang-serve-spa)

```bash
# clone repo
$ git clone git@github.com:cage1016/golang-serve-spa.git

# npm install package
$ npm install

# 一鍵建立 docker image
$ npm docker

# docker run
$ docker run -d --name counter -p 3000:3000 username/counter:v0.1.0
```

