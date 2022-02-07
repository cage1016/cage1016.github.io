# Screenshot as a Service


<!--more-->

[GCPUG Taiwan Meetup #29](https://gcpugtw.kktix.cc/events/meetup201709) Screenshot as a service

> [cage1016/screenshot-as-a-service-demo: GCPUG Taiwan Meetup #29: screenshot as a service demo code](https://github.com/cage1016/screenshot-as-a-service-demo)

{{< slideshare id="79563683" >}}

### Waldo-gcp

> 一個基本 Google Map 上計算最佳路徑的實小服務

{{<image src="img/screenshot-as-a-service-1.png">}}

當初會想弄一個 screenshot 的服務是因為 [slideshare: Waldo-gcp](https://goo.gl/fnqLaZ) 中會有擷圖的需求，且需要跟 GAE 進行整合，所以選擇了滿多人使用的 [PhantomJS](https://goo.gl/DT28P) 來實作擷圖的服務並使用 GAE flex runtime 的型式發佈到 GAE 平台上作為專案的 microservice 使用

### Phantomjs

> PhantomJS is a headless WebKit scriptable with a JavaScript API. It has fast and native support for various web standards: DOM handling, CSS selector, JSON, Canvas, and SVG.
> http://phantomjs.org/

```js
// yahoo.com.tw.js
var page = require('webpage').create();
page.viewportSize = { width: 1440, height: 900 };
page.clipRect = { top: 0, left: 0, width: 1440, height: 900 };
page.open('http://yahoo.com.tw', function() {
  page.render('yahoo.com.tw.png');
  phantom.exit();
});

// execute
$ phantomjs yahoo.com.tw.js
```

### chromless

> Chrome automation made simple. Runs locally or headless on AWS Lambda.
> https://github.com/graphcool/chromeless

```js
const { Chromeless } = require('chromeless')

async function run() {
  const chromeless = new Chromeless()

  const screenshot = await chromeless
    .goto('https://www.google.com')
    .type('chromeless', 'input[name="q"]')
    .press(13)
    .wait('#resultStats')
    .screenshot()

  console.log(screenshot) // prints local file path or S3 url

  await chromeless.end()
}

run().catch(console.error.bind(console))
```

### puppeteer

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://www.google.com');
  await page.screenshot({path: 'google.com.png'});

  browser.close();
})();
```

這兒舉三個簡單的例子來說明擷圖的技術，Phantomjs, Chromeless, puppeteer, 以前 Phantomjs 很熱門，不過最近 chromeless 等技術興起，也有友好的 API 可以給開發人員使用。[GoogleChrome/puppeteer: Headless Chrome Node API](https://goo.gl/cz4fSi) 更是提供高階 API 來操作 [Headless](https://goo.gl/wg3u1W) Chrome

### Screenshot as service via GAE custom runtime

_app.yaml_

```yaml
runtime: custom
env: flex
service: screenshot

...

handlers:

- url: /.*
  script: app.js
```

由於在兒我們使用的是包了 [Express](https://goo.gl/Ro4G) 及 [PhantomJS](http://phantomjs.org/) 的 custome runtime 來實作 GAE 版的 screenshot service, `yaml` 的重點便是指定 `runtime: custom`, `env: flex`

_dockerfile_

```dockerfile
FROM launcher.gcr.io/google/debian8

RUN DEBIAN_FRONTEND=noninteractive apt-get update -y && apt-get install --no-install-recommends -y -q curl apt-utils build-essential ca-certificates libfreetype6 libfontconfig1

RUN echo "deb http://http.debian.net/debian jessie-backports main" > /etc/apt/sources.list.d/backports.list && \
    apt-get update -y && \
    apt-get install -y --no-install-recommends fonts-noto fonts-noto-cjk locales-all && \
    apt-get clean && \
    apt-get autoclean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir /nodejs && curl http://nodejs.org/dist/v0.12.1/node-v0.12.1-linux-x64.tar.gz | tar xvzf - -C /nodejs --strip-components=1

ENV PATH $PATH:/nodejs/bin
WORKDIR /app
ADD package.json /app/
RUN npm install
ADD . /app

ENTRYPOINT ["/nodejs/bin/npm", "start"]
```

`DockerFile` 的重點則是準備好基本的 library 及 Express/Phantomjs 的配置。準備好 `DockerFile`，`app.yaml` 還有程式碼，就可以透過 `gcloud -q app deploy` 的指命將服務推到 GCP 上

{{<image src="img/screenshot-as-a-service-2.png">}}

> https://waldo-gcp-testbed.appspot.com/screenshot/usage.html
