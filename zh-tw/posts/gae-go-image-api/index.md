# GAE go Image API for GCS


<!--more-->

> [LIVE DEMO](https://go-gae-image-api-example-dot-gae-lab-001.appspot.com/)

在開發 GAE 應用程式時，難免會遇到應用程式需要處理圖片的問題。GAE 的應用程用可以直接存取靜態的資源檔案，這塊直接在 `app.yaml` 檔案中設定就可以了，不過也因為 GAE 應用程式的特性，需要將所有的資源上傳一份到 GAE 正式環境中，所以會發現上傳專案的容量大小會爆增(如果你將所有需要的圖檔階直接放在程式內)。

專案檔案太大會影響 GAE 在自動擴展的效能，所以盡可能的將不必要的東西移多專案外(圖檔等)


_app.yaml_

```yaml
...


- url: /favicon.jpg
  mime_type: image/x-icon
  static_files: public/images/favicon.jpg
  upload: public/images/favicon.jpg
 
- url: /js/*
  mime_type: text/javascript
  static_dir: public/js
  secure: always
 
- url: /css/*
  mime_type: text/css
  static_dir: public/css
  secure: always
  http_headers:
    Access-Control-Allow-Origin: "*"
 
- url: /images/*
  static_dir: public/images
  secure: always
```

每一個 GCP 的專案可以有 `5G` 免費的 Google Cloud Storage 空間，所以將 GAE 上大檔案的靜態資源放到 GCS 是一個很好的決定，在 GAE 程式中可以使用 `google-api-go-client` 來存取 GCS 上相關的資源。如果是圖片檔的話，更可以直接用 Image API 的方式來直接讀取圖片資源

[Images Go API](https://cloud.google.com/appengine/docs/go/images/) 讓我們使用 `Blobstore` 的方式來讀取/裁切圖檔，當我們透過 Blobstore 拿到靜態圖檔時可以直接透過 url 的方式對目標圖片進行調整圖檔的大小或進行裁切

```
# 將目標圖片重新縮放至 32 像素(等比例 - 長邊)
http://lhx.ggpht.com/randomStringImageId=s32
 
# 將照片裁切至 32 像素
http://lhx.ggpht.com/randomStringImageId=s32-c
```

## imgServe

因些我們就可以設計一個動態的路由來對應至 GCS 上特定路徑的圖片

```
/img?f=dress/banner.png --> GCS:Bucket/dress/banner.png
/img?f=dress/color/banner.png --> GCS:Bucket/dress/color/banner.png
/img?f=dress/color/banner.png&s=200 --> GCS:Bucket/dress/color/banner.png 等比縮放至 200 像素
/img?f=dress/color/banner.png&s=200-c --> GCS:Bucket/dress/color/banner.png 等比縮放至 200 像素且進行 1:1 的裁切
```

如此一來前端面頁如果需要引用照片，可以直接使用 `<img src="/img?f=dress/banner.png" alt="">`，如果目標圖片不存在則直接輸入一開始就準備好的 image 404 圖片進行代替

_main.go_

```go
package main
 
import (
	"fmt"
	"golang.org/x/net/context"
	"net/http"
 
	"github.com/labstack/echo"
	"github.com/labstack/echo/engine/standard"
 
	"google.golang.org/appengine"
	"google.golang.org/appengine/blobstore"
	"google.golang.org/appengine/image"
)
 
const (
	Bucket           = "<your-gae-default-bucket>"
	NotFoundImageURL = "img-api-example/img404.jpg"
)
 
/**
 * index page
 */
func indexGET(c echo.Context) error {
	c.Render(200, "index.tpl", map[string]interface{}{
		"title": "go gae image API example",
		"images": []string{
			"img?f=img-api-example/1.png",
			"img?f=img-api-example/1.png&s=200",
			"img?f=img-api-example/1.png&s=200-c",
			"img?f=img-api-example/2.jpg",
		},
	})
	return nil
}
 
/**
 * get not foound image default image, img404.jpg
 */
func getNotFoundImage(ctx context.Context) (urlString string, err error) {
	blobPath := fmt.Sprintf("/gs/%s/%s", Bucket, NotFoundImageURL)
	blobKey, err := blobstore.BlobKeyForFile(ctx, blobPath)
	if err != nil {
		return
	}
	opts := image.ServingURLOptions{Secure: false, Crop: true}
	url, err := image.ServingURL(ctx, blobKey, &opts)
	if err != nil {
		return
	}
	urlString = url.String()
	return
}
 
/**
 * image serve handler
 */
func imgServe(c echo.Context) error {
	ctx := appengine.NewContext(c.Request().(*standard.Request).Request)
 
	filePath := c.QueryParam("f")
	size := c.QueryParam("s")
 
	blobPath := fmt.Sprintf("/gs/%s/%s", Bucket, filePath)
	blobKey, err := blobstore.BlobKeyForFile(ctx, blobPath)
	if err != nil {
		c.String(http.StatusExpectationFailed, err.Error())
	}
 
	opts := image.ServingURLOptions{Secure: false, Crop: true}
	if url, err := image.ServingURL(ctx, blobKey, &opts); err != nil {
		if n, err := getNotFoundImage(ctx); err != nil {
			return err
		} else {
			c.Redirect(http.StatusTemporaryRedirect, n)
		}
	} else {
		if size != "" {
			c.Redirect(http.StatusTemporaryRedirect, fmt.Sprintf("%s=s%s", url.String(), size))
		} else {
			c.Redirect(http.StatusTemporaryRedirect, url.String())
		}
	}
 
	return nil
}
 
func init() {
	e.GET("/", indexGET)
	e.GET("/img", imgServe)
}
```

### GAE Go Image serve GCS image
{{<image src="gae-go-image-api.png">}}

### demo code

[cage1016/gae-go-image-example: GAE go image API example](https://github.com/cage1016/gae-go-image-example)

