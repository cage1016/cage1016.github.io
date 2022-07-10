# Alfred Open Chinese Convert 開放中文轉換


<!--more-->

## 原由

由於 Mac OS 在 `12.3` 終於移除對 Python2 的支援，導至 Alfred 4 版本中原先使用的 Python2 編寫的 workflow 都不能運作了。又因為工作需要會有簡繁轉換的需求，之前使用的 [amowu/alfred-chinese-converter](https://github.com/amowu/alfred-chinese-converter) 作者也沒有再更新了 (停留在 Alfred2)
。所以透過 Golang + [deanishe/awgo](https://github.com/deanishe/awgo) 自己造一個也省去了對 Python 的依賴, Alfred 升級也不會再遇到 Python 版本的問題。

## alfred-opencc

- 預設使用 `occ` 關鍵字啟動

    {{<image src="/posts/alfred-open-chinese-convert/img/1.jpg" alt="alfred-opencc 預設使用 `occ` 關鍵字啟動">}}

- 支援 `8` 種轉換方式，可以分別啟用/停用轉換方式

    1. 簡體到繁體
    1. 繁體到簡體
    1. 簡體到臺灣正體
    1. 臺灣正體到簡體
    1. 簡體到香港繁體
    1. 香港繁體到簡體
    1. 簡體到繁體（臺灣正體標準）並轉換爲臺灣常用詞彙
    1. 繁體（臺灣正體標準）到簡體並轉換爲中國大陸常用詞彙

    {{<image src="/posts/alfred-open-chinese-convert/img/placeholder.jpg" alt="alfred-opencc 支援 8 種轉換方式">}}


下載方式，訪問 [Releases · cage1016/alfred-opencc](https://github.com/cage1016/alfred-opencc/releases) 直接下載最新版本安裝即可
