# Alfred Open Chinese Convert


<!--more-->

## Introduction

Since Mac OS finally removed support for Python2 in `12.3`, the workflows written in Python2 in Alfred 4 are no longer available. The [amowu/alfred-chinese-converter](https://github.com/amowu/alfred-chinese-converter) I was using before is also no longer updated (it's still in Alfred 2). So I built one with Golang + [deanishe/awgo](https://github.com/deanishe/awgo) + [longbridgeapp/opencc](https://github.com/longbridgeapp/opencc), eliminating the dependency on Python, and Alfred upgrades no longer run into Python version problems.

## alfred-opencc

- default keyword `occ` launch alfred-opencc

    {{<image src="/posts/alfred-open-chinese-convert/img/1.jpg" alt="adefault keyword `occ` launch alfred-opencc">}}

- Total support `8` translation methods. Be able to enable/disable for each translation method indvidual.

    1. Simplified Chinese to Traditional Chinese
    1. Traditional Chinese to Simplified Chinese
    1. Simplified Chinese to Traditional Chinese (Taiwan Standard)
    1. Traditional Chinese (Taiwan Standard) to Simplified Chinese
    1. Simplified Chinese to Traditional Chinese (Hong Kong variant)
    1. Traditional Chinese (Hong Kong variant) to Simplified Chinese
    1. Simplified Chinese to Traditional Chinese (Taiwan Standard) with Taiwanese idiom
    1. Traditional Chinese (Taiwan Standard) to Simplified Chinese with Mainland Chinese idiom

    {{<image src="/posts/alfred-open-chinese-convert/img/placeholder.jpg" alt="alfred-opencc support 8 translation methods">}}


Please visit [Releases · cage1016/alfred-opencc](https://github.com/cage1016/alfred-opencc/releases) to download latest version alfred-opencc workflow
