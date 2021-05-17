# notebook - golang regexp


<!--more-->

{{<image src="img/go-regexp-notebook-1.png">}}

簡單記鎑一下最近在專案上需要利用 regex 來進行字串的拆解

```yaml
cfg:/etc/config/a.conf = 0
Build=20160331
Version = 2.2.13
Date = 2016-06-28
RC_Number = 181
Enable=
Status = complete
ePassword = V2@W5Q9N91N4fXGEEyL+yXOlw==
Server Type = 5
Check External IP = 10
Enable = TRUE
```

需要將將上述的 conf 拆分成 key, value，意外發現了一個很常好用的網站 [Online regex tester and debugger: PHP, PCRE, Python, Golang and JavaScript](https://regex101.com/)，可以很容易的在線上即時的進行測試，更棒的地方是他還是可以產出程式碼

```go
package main

import (
    "regexp"
    "fmt"
)

func main() {
    var re = regexp.MustCompile(`([^=]*)=(.*)`)
    var str = `cfg:/etc/config/a.conf = 0
Build=20160331
Version = 2.2.13
Date = 2016-06-28
RC_Number = 181
Enable=
Status = complete
ePassword = V2@W5Q9N91N4fXGEEyL+yXOlw==
Server Type = 5
Check External IP = 10
Enable = TRUEÅ`
    
    for i, match := range re.FindAllString(str, -1) {
        fmt.Println(match, "found at index", i)
    }
}
```

### Reference
- [StefanSchroeder/Golang-Regex-Tutorial: Golang - Regular Expression Tutorial](https://github.com/StefanSchroeder/Golang-Regex-Tutorial)
- [基础知识 - Golang 中的正则表达式 - GoLove - 博客园](http://www.cnblogs.com/golove/p/3269099.html)
- [Online regex tester and debugger: PHP, PCRE, Python, Golang and JavaScript](https://regex101.com/)
