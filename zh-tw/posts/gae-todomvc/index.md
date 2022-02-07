# GAE-todomvc


<!--more-->

最近需要幫內部基於[GAE - Python](https://cloud.google.com/appengine/docs/python/)平台上導入前端的框架，[TodoMVC](http://todomvc.com/)
是一個非常適合拿來學習前端框架的資源，它以**TodoMVC**的題目實作目前主流的前端框架(**React**、**Angular**、**Vuejs**、**Ember.js**、**Polymer** 等等)，
你可以看到不同框架的優缺點，選擇一個最適合你的框架來學習。

在 [cage1016/gae-todomvc](https://github.com/cage1016/gae-todomvc) 中則選用了 `Reactjs (Flux)`、`AnguarJs`、`Vue.js` 三個前端框架來搭配 GAE-Python + Datastore + Endpoints APIs。

### Spec

- Front-end: `Reactjs (Flux)`、`AnguarJs`、`Vue.js`
- back-end: `GAE-Python (webapp2)` + `Datastore` +　`Endpoints APIs`

### Getting Started

三個前端框架的程式碼都是基於 [TodoMVC](http://todomvc.com/) 版本 clone 後稍作修改 (以符合 Endpoints RESTFul APIs)

- [todomvc/examples/angularjs-perf at gh-pages · tastejs/todomvc](https://github.com/tastejs/todomvc/tree/gh-pages/examples/angularjs-perf)
- [flux/examples/flux-todomvc at master · facebook/flux](https://github.com/facebook/flux/tree/master/examples/flux-todomvc/)
- [todomvc/examples/vue at gh-pages · tastejs/todomvc](https://github.com/tastejs/todomvc/tree/gh-pages/examples/vue)

GAE todomvc 的 gcloud SDK 為 `0.9.64`

```shell
# Get gcloud
$ curl https://sdk.cloud.google.com | bash

# Get App Engine component
$ gcloud components update app
$ gcloud components update gae-python

# Clone repo from github
$ git clone https://github.com/cage1016/gae-todomvc

# Install pip packages
$ sudo pip install -r requirements.txt -t lib

# Install npm packages
$ npm install

# Install bower packages
$ bower install
```

GAE todomvc 中 `Vue.js` 範例中使用到了 `vue-resource` library，因為 `vue-resource` 模組預設沒有 `update: {method: 'put'}` method，所以在執行 `gulp` 時，需自己稍作修改。

```shell
# switch to bower_components
$ cd bower_components

# clone vue-resource repo from github
$ git clone https://github.com/vuejs/vue-resource

# switch to vue-resource folder
$ cd vue-resource

# install vue-resource require packages
$ npm install

# add update method
# /bower_components/vue-resource/src/resource.js
# add "update: {method: 'put'}" at line 109

# rebuild vue-resource
$ npm run build

# go back
$ cd ../..

# Build
$ gulp

# Run GAE locally
$ dev_appserver.py app.yaml
```

瀏覽 [http://localhost:8080](http://localhost:8080) 即可以看到結果。

### Screencapture

{{<image src="img/gae-todomvc-1.png">}}

### GAE Endpoints APIs

GAE Endpoints APIs 詳細的使用方式可以參考 [Creating an Endpoints API](https://cloud.google.com/appengine/docs/python/endpoints/create_api)，
GAE-todomvc 則是使用 [GoogleCloudPlatform/endpoints-proto-datastore](https://github.com/GoogleCloudPlatform/endpoints-proto-datastore) 來直接存取 Datastore 的 Model。另外 GAE Endpoints APIs 也提供了本地開發測試的Url [http://localhost:8080/_ah/api/explorer](http://localhost:8080/_ah/api/explorer)

### Reference
- [TodoMVC](http://todomvc.com/)
- [Web Starter Kit — Web Fundamentals](https://developers.google.com/web/tools/starter-kit/)
- [[React] 資源整理 « Huli's Blog](http://huli.logdown.com/posts/276040-react-resource-consolidation)
- [React & Flux Workshop](http://www.slideshare.net/xmlilley/react-flux-50660816)
- [Reactjs-JQuery-Vuejs-Extjs-Angularjs对比 - 【当耐特】 - 博客园](http://www.cnblogs.com/iamzhanglei/p/4481521.html)
- [深入浅出React（一）：React的设计哲学 - 简单之美](http://www.infoq.com/cn/articles/react-art-of-simplity)
- [深入浅出React（二）：React开发神器Webpack](http://www.infoq.com/cn/articles/react-and-webpack?utm_source=infoq&utm_medium=related_content_link&utm_campaign=relatedContent_articles_clk)

