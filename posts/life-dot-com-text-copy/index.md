# Life.com text copy


Facebook 許多人都有轉載一些文章，看到有些不錯的文章，會想保存到 Evernote 上，Evernote 提供了 [Evernote Web Clipper - Chrome Web Store](https://chrome.google.com/webstore/detail/evernote-web-clipper/pioclpoplcdbaefihamjohnefbikjilc) 及 [Clearly - Chrome Web Store](https://chrome.google.com/webstore/detail/clearly/iooicodkiihhpojmmeghjclgihfjdjhj) 可以方便使用直接把面頁的文章快速的存到自己的 Notebook 中。

[life.com](http://www.life.com.tw) 是網友滿常轉載的一個媒體之一。不過後來發現 Life.com 把文章內容的**選取**及**複製**功能都關閉了。

查看原始碼發現文章內容被包在**iframe**中

```html
<iframe src="about:blank" frameborder="0" border="0" cellspacing="0" style="width: 600px; border: 0px; height: 4887px;">
  <html>
    <head>
      <style type="text/css">...</style>
    </head>
    <body>
      ....
    </body>
  </html>
</iframe>
```

### 快速的解決方法
由於網頁本身有戴入 **jQuery**，所以在 Chrome 的 console 中直接執行

```javascript
// enable iframe select and contextmenu feature.
jQuery('iframe[src="about:blank"]')[0].contentDocument.oncontextmenu = function(){return true;}
jQuery('iframe[src="about:blank"]')[0].contentDocument.onselectstart = function(){return true;}
```

以上的方法雖然可以解決選取及複製，不過 Evernote-web-clipper 及 Clearly 還是沒有讀取 iframe 中的內容。

### Chrome extension 解決方法
一勞永逸的方式就是透過程式把 iframe 中的內容 unwrap 出來 (Trick: 前提是 iframe 中 src="about:blank"，iframe 有其他的限制)。

#### 原來的 html

```html
<div aricle-detail-main="" class="aricle-detail-mainin" id="mainContent">
  <iframe src="about:blank" frameborder="0" border="0" cellspacing="0" style="width: 600px; border: 0px; height: 4887px;">
  <html>
    <head>
      <style type="text/css">...</style>
    </head>
    <body>
      ....
    </body>
  </html>  
  </iframe>
</div>
```

#### unwrap 後的 html，iframe 中的 style 則寫到 head 中。

```html
<div aricle-detail-main="" class="aricle-detail-mainin" id="mainContent">
  <div id="life-dot-com-copy">
    ...原來 iframe 中 body 的內容...
  </div>
</div>
```

#### 結果
Evernote-web-clipper 及 Clearly 都可以正常的讀到內容。

{{<image src="img/life-dom-copy-text-1.png">}}

{{<image src="img/life-dom-copy-text-2.png">}}

### Installing
[life.com text copy - Chrome Web Store](https://chrome.google.com/webstore/detail/lifecom-text-copy/oelpalillkokjbeojomcpkafgoelilbk?hl=en&gl=TW)

### Contribute

```shell
# Clone repo from github
$ git clone https://github.com/cage1016/life.com-text-copy

# Install npm package
$ npm install

# debug
$ grunt debug

# build extension
$ grunt
```

