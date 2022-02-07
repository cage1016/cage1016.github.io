# webpack es6 demo


<!--more-->

最近在[InfoQ](http://www.infoq.com/cn/) 中看到一篇文章 [成為一名優秀的Web前端開發者](http://goo.gl/MUBDUV)，裡面提到學習 [ECMAScript 2015](http://www.infoq.com/news/2015/06/ecmascript-2015-es6) 這一個部份，又因為在 github 看到越來越多 Javascript 專案始用 **ES6**

> Javascript這個語言，是有「標準規格」的，叫做ECMA-262，這個標準規格，是由ECMA International (European Computer Manufacturers Association International)這個標準組織所制定，所以也稱呼它為ECMAScript。這個規格涵蓋了所有Javascript的核心，是各種Javascript執行環境一定會有的東西。在此之上，瀏覽器內就會額外加入DOM等擴充，而像node.js這樣的伺服器端環境，也有他自己的擴充，不過核心的部份是大家都一樣的。（附帶一提，C#的標準規格也是ECMA制定的） [初探 ES6（1）Harmony 的黑歷史 by fillano | CodeData](http://www.codedata.com.tw/javascript/introducing-es6-1-harmony-history/)

2015 年的 ECMAScript 是一個顯著的更新。上一次版本 **ES5** 的標準是在 2009 年訂定的，AngularJS, Aurelia, ReactJS, Ionic 都是架構在 **ES5** 的基礎上， 不過 ReactJS 也開始支援了 **ES6**，身為一個前端的工程師也是時候開始學習了

# ECMAScript 2015 (**ES6**)
**ES6** 包含了很多的新特性，更詳細的說明請參照 [Reference - ES6](./{{< relref "#es6" >}})，本次的 Simple 只用到幾個 **ES6** 最基本用法

- arrow
- classes
- enhanced object literals
- template strings
- destructuring
- default + rest + spread
- let + const
- iterators + for..of
- generators
- unicode
- modules
- module loaders
- map + set + weakmap + weakset
- proxies
- symbols
- subclassable built-ins
- promises
- math + number + string + object APIs
- binary and octal literals
- reflect api
- tail calls

# Setting Started

```shell
# clone webpack-es6 repo
$ git clone https://github.com/cage1016/webpack-es6-demo

# install node package
$ npm install

# complie js
$ npm run watch

# open index.html
$ open -a 'Google Chrome' index.html
```

## webpack
目前 Browser 還沒有全面支援 **ES6**，所以在使用時候就必需進行轉譯，這邊我們使用的是 [shama/es6-loader](https://github.com/shama/es6-loader) 的轉譯器

_webpack.config.js_

進行 webpack.config.js 檔案的配置:

1. 設置 webpack 進入點. `./app/entry.js`
2. 設置轉譯後檔案 `bundle.js` (最後需要在 `index.html` 中引用 `bundle.js`)
3. 設置 `babel-loader` 模組來進行 **ES6** 的轉譯

```javascript
var path = require('path');
var webpack = require('webpack');

module.exports = {
  entry: './app/entry.js',
  output: {
    path: __dirname,
    filename: 'bundle.js',
  },
  module: {
    loaders: [{
      test: path.join(__dirname, 'app'),
      loader: 'babel-loader',
      exclude: /node_modules/,
    }, ],
  },
  plugins: [
    // Avoid publishing files when compilation failed
    new webpack.NoErrorsPlugin(),
  ],
  stats: {
    // Nice colored output
    colors: true,
  },
  // Create Sourcemaps for the bundle
  devtool: 'source-map',
};
```

## Polygon

_app/Polygon.js_

在 `app/Polygon.js` 中使用到了 **ES6** 中三個新的特性:

1. classes/inheritance/static
2. module
3. arrow
4. strings

```javascript
class Polygon {
  constructor(points) {
    this.points = points;
  }

  getArea() {
    let area = 0;
    let j = this.points.length - 1;

    for (let i = 0; i < this.points.length; i++) {
      area = area +
        (this.points[j][0] + this.points[i][0]) *
        (this.points[j][1] - this.points[i][1]);
      j = i;
    }

    return Math.abs(area / 2);
  }

  getPoints() {
    return this.points.map(n => `(${n[0]},${n[1]})`).join(',');
  }

  toString() {
    return `I am ${this.name}, area = ${this.getArea()}`
  }
}

class Rectangle extends Polygon {
  constructor(points) {
    super(points, 'Rectanger');
    this.name = 'Rectanger'
  }
}

class Triangle extends Polygon {
  constructor(points) {
    super(points, 'Triangle');
    this.name = 'Triangle';
  }

  static create(points) {
    return new Triangle(points);
  }
}

export {
  Triangle, Polygon, Rectangle
}
```

## Entry

_app/entry.js_

在 `app/entry.js` 中使用到了 **ES6** 中新的特性:

1. module 引用的方式
2. let (類似 **ES5** 中的 `var`，不過 let 會把 scope 區隔開來)

```javascript
import {
  Rectangle, Triangle
}
from './Polygon';

let r = new Rectangle([
  [4, 4],
  [4, -4],
  [-4, -4],
  [-4, 4],
]);

let t = Triangle.create([
  [4, 6],
  [4, -4],
  [8, -4],
])

let body = document.querySelector('body');
body.innerHTML = [
  r.toString(),
  r.getPoints(),
  t.toString(),
  t.getPoints(),
].join('<br/>');

```

最後在 Browser 中看到三結果

```html
I am Rectanger, area = 64
(4,4),(4,-4),(-4,-4),(-4,4)
I am Triangle, area = 20
(4,6),(4,-4),(8,-4)
```

# Reference
## ES6
- [ECMAScript 6入门](http://es6.ruanyifeng.com/)
- [es6-guide - GitBook](https://www.gitbook.com/book/mrzepinski/es6-guide/details)
- [ES6 In Depth: An Introduction ✩ Mozilla Hacks – the Web developer blog](https://hacks.mozilla.org/2015/04/es6-in-depth-an-introduction/)
- [Classes - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

## webpack
- [tutorials/getting-started](http://webpack.github.io/docs/tutorials/getting-started/)
- [shama/es6-loader](https://github.com/shama/es6-loader)
- [Babel · The compiler for writing next generation JavaScript](https://babeljs.io/)

