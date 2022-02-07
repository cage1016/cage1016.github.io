# iOS ringtone maker from YouTube video


<!--more-->

最近大竹弟打電話問說 iOS 鈴聲在 Garageband 上面製作的檔案怎麼這麼大，其實只有 5 秒的鈴聲

先來說一下 iPhone 上配置客製的鈴聲有二種方式

- iTunes
- Garageband 上編輯直接輸出至鈴聲

由於 macOS 10.15 版本就沒有 itunes 可以用，所以基本上網路的教學大也是直接在 Garageband 上直接編輯後直接輸出至 iPhone 的鈴聲

以前我自己也是換過不少鈴聲，以前是透過 [Free Ringtones for Android and iPhone. Free Ringtone Maker - Audiko](https://sur.ly/o/audiko.tw/AA000014) 選擇喜歡的檔案下載安裝

雖然現成的工具方便，不過有時候找不到自己喜歡的歌曲，那就土砲一個吧

### iOS ringtone maker from YouTube video

[cage1016/ios-ringtone-maker-yt: iOS ringtone maker from Youtube video](https://github.com/cage1016/ios-ringtone-maker-yt)

基本上的流程

- 找到喜歡的 YouTube vide
- 找到想要音頻的開始秒數 ex: 00:04:00
- 決定鈴聲的秒數 ex: 30
- 是否要音頻漸入漸出效果
- 直接下 command container 會直接輸出 m4r
- copy 至 iCloud
- 在 Garageband 引入 m4r
- 輸出至鈴聲
- 打完收工

### demo

1. 找到一個 YouTube 影片 [Señorita - YouTube](https://www.youtube.com/watch?v=Pkh8UtuejGw)
1. 從 22 秒開始
1. 共 30 秒
1. 音頻漸入漸出效果: fadein 3 秒， fadeout 3 秒

{{< asciinema rows="20" key="414953" start-at="5" loop="true" >}}

鈴聲直接輸出在執行 docker 路徑下 `./output` 

```shell
$ ls ./output/
Pkh8UtuejGw.m4r
```

剩下的就是將檔案傳至手機，透過 Garageband 輸出至 iPhone

DONE!

BTW. 有時候再來弄 UI

{{< music url="Pkh8UtuejGw.m4r" name="Señorita" artist="Shawn Mendes" cover="cover.jpeg" >}}
