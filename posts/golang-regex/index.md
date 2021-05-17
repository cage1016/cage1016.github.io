# Golang regEx parse Facebook Live rtmp


<!--more-->

最近因為公司專案的關係，開始接觸 Facebook Live API，需要動態的透過 API 去建立一組 Live 直播 並將 rtmp 的整個字串拆解成 `ServerUrl` 及 `StreamKey` 二個部份再由 ffmpeg 拿到這個資訊去 streaming。因為 Youtube Data API v3 可以別分拿 `StreamName` 及 `IngestionAddress`, 所以只需要對 Facebook 的部份特別處理


先來看一下 Facebook Live API 建立直播拿回來的完整的 rmtp url

```
rtmp://rtmp-api.facebook.com:80/rtmp/1854721994809041?ds=1&s_l=1&a=ATjdLqx4xd23Q2mF
```

完整的 rtmp url 由 StreamUrl + StreamKey 組成

- StreamUrl: `rtmp://rtmp-api.facebook.com:80/rtmp/`
- StreamKey: `1854721994809041?ds=1&s_l=1&a=ATjdLqx4xd23Q2mF`

因此可以透過 Golang regEx 的方式來拆解

```go
package main

import (
    "fmt"
    "regexp"
)


func getParams(regEx, url string) (paramsMap map[string]string) {
    var compRegEx = regexp.MustCompile(regEx)
    match := compRegEx.FindStringSubmatch(url)

    paramsMap = make(map[string]string)
    for i, name := range compRegEx.SubexpNames() {
        if i > 0 && i <= len(match) {
            paramsMap[name] = match[i]
        }
    }
    return
}

func main()  {
    params := getParams(`(?P<ServerKey>^rtmp://.+/)(?P<StreamKey>.+)`, `rtmp://rtmp-api.facebook.com:80/rtmp/1854721994809041?ds=1&s_l=1&a=ATjdLqx4xd23Q2mF`)
    fmt.Println(params["ServerKey"]) // rtmp://rtmp-api.facebook.com:80/rtmp/
    fmt.Println(params["StreamKey"]) // 1854721994809041?ds=1&s_l=1&a=ATjdLqx4xd23Q2mF
}
```

[The Go Playground - Demo](https://goo.gl/7nSGim)

# reference

1. [Regex Tester Golang - A Go regular expression online tester](https://regex-golang.appspot.com/assets/html/index.html)
2. [Facebook Live API - Video](https://developers.facebook.com/docs/videos/live-video)

