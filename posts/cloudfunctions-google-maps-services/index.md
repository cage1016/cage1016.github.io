# CloudFunctions Google maps service


<!--more-->

### Waldo-gcp

[Waldo-gcp](https://www.slideshare.net/cagechung/waldogcp) 在 2015 時 Google I/O Extended Taipei 時分享過一個計算最佳路徑的微服務。在提供幾組 Google Maps 上有效的地址，透過 Google Maps Distance Matrix API 來計算出每一個點一點之間的旅行距離及旅行時間。再透過基因演算出計算出周遊一圈旅行最短路徑

```shell
# 提供5組 Google Maps 上有效的地址

台北市內湖區瑞光路227號1樓,
高雄市鼓山區美術東二路106號,
台南市長榮路一段175號,
臺北市松山區南京東路五段123巷1弄15號,
高雄市五福四路131號2樓
```

```shell
# 透過 Google Maps Distance Matrix API 計算出  origin, destination, distance, duration

台北市內湖區瑞光路227號1樓, 高雄市鼓山區美術東二路106號, 352777, 13532
台北市內湖區瑞光路227號1樓, 台南市長榮路一段175號, 314041, 12166
台北市內湖區瑞光路227號1樓, 臺北市松山區南京東路五段123巷1弄15號, 5167, 783
台北市內湖區瑞光路227號1樓, 高雄市五福四路131號2樓, 356215, 14018
高雄市鼓山區美術東二路106號, 台南市長榮路一段175號, 46668, 3203
高雄市鼓山區美術東二路106號, 臺北市松山區南京東路五段123巷1弄15號, 355397, 14131
高雄市鼓山區美術東二路106號, 高雄市五福四路131號2樓, 4166, 748
台南市長榮路一段175號, 臺北市松山區南京東路五段123巷1弄15號, 314250, 12475
台南市長榮路一段175號, 高雄市五福四路131號2樓, 49930, 3339
臺北市松山區南京東路五段123巷1弄15號, 高雄市五福四路131號2樓, 359485, 14338
```

{{<image src="img/cloudfunctions-google-maps-services-1.png">}}

### CloudFunctions Google maps service

> Google Cloud Functions is a lightweight compute solution for developers to create single-purpose, stand-alone functions that respond to Cloud events without the need to manage a server or runtime environment - Google Cloud Functions Documentation

[Waldo-gcp](https://www.slideshare.net/cagechung/waldogcp) 微服務中透過 flex runtime GAE Python 中的 library(googlemaps) 來呼叫 Google Maps Distance Matrix API. Google Function 在這很適合取代原來 Python 版本的 Google Maps Distance Matrix API. 

Cloud functions 可以建立 `Background Funtions`, `HTTP Functions` + `HTTP Triggers`, `Cloud Pub/Sub Triggers`, `Cloud Storage Triggers`, `Direct Triggers`. 所以我們可以建立一個可以執行 Google Maps Distance Matrix API 的 Cloud Functions, 透過起 `HTTP Triggers`, `Direct Triggers` 即可以建立計算旅行路徑的距離及時間

## Getting Started

Github repo: [cage1016/cloudfunctions-google-maps-services](https://github.com/cage1016/cloudfunctions-google-maps-services)

```shell
# clone repo
$ git git@github.com:cage1016/cloudfunctions-google-maps-services.git && cd cloudfunctions-google-maps-services

# install node packages
$ npm install
```

修改 `index.js` 中 API Key, `<YOUR-GCP-API-KEY>`, 並確保你的 GCP 專案有啟用 Google Maps Distance Matrix API

```js
const googleMapsClient = require('@google/maps').createClient({
    key: '<YOUR-GCP-API-KEY>'
})

...
```

### distanceMatrix background

> 修改 `makefile` 中 `BUCKET=<YOUR-GCS-BUCKET>` Google Cloud Storage Bucket, Bucket 為 Cloud Functions 上傳的位置

修改完依序執行即可

```shell
# deploy distanceMatrix
$ make deploy_backend

# call distanceMatrix
$ make call_backend

# log distanceMatrix
$ make log_backend

# show description of distanceMatrix function
$ make describe_backend
```

### distanceMatrix background

> 修改 `https://<YOUR_REGION>-<YOUR_PROJECT_ID>.cloudfunctions.net/distanceMatrixHttp` endpoint, endpoints 可由  `make deploy_http` 中得到

修改完依序執行即可

```shell
# deploy distanceMatrixHttp
$ make deploy_http

# call distanceMatrixHttp
$ make call_http

# log distanceMatrixHttp
$ make log_http

# show description of distanceMatrixHttp function
$ make describe_http
```

### CloudFunctions

{{<image src="img/cloudfunctions-google-maps-services-2.png">}}

## Reference
- [googlemaps/google-maps-services-js: Node.js client library for Google Maps API Web Services](https://github.com/googlemaps/google-maps-services-js)
- [Developer's Guide  |  Google Maps Distance Matrix API  |  Google Developers](https://developers.google.com/maps/documentation/distance-matrix/intro?hl=en)
- [Google Cloud Functions Documentation  |  Cloud Functions  |  Google Cloud Platform](https://cloud.google.com/functions/docs/)

