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

```shell
$ docker run --rm -it -v $(PWD)/output:/app/output -e FADEIN=3 -e FADEOUT=3 -e VID=Pkh8UtuejGw -e SS=00:00:22 -e T=30 cage1016/ios-ringtone-maker-yt:latest
YoutTubeID: Pkh8UtuejGw
StartTime: 00:00:22
Duration: 30
FadeIn: 3
FadeOut: 3

 Site:      YouTube youtube.com
 Title:     Shawn Mendes, Camila Cabello - Señorita
 Type:      video
 Stream:
     [140]  -------------------
     Quality:         audio/mp4; codecs="mp4a.40.2"
     Size:            3.17 MiB (3325319 Bytes)
     # download with: annie -f 140 ...

 3.17 MiB / 3.17 MiB [==============================================================================================================================================================================] 100.00% 147.17 KiB/s 22s
ffmpeg version 4.1.5 Copyright (c) 2000-2020 the FFmpeg developers
  built with gcc 9.2.0 (Alpine 9.2.0)
  configuration: --disable-debug --disable-doc --disable-ffplay --enable-shared --enable-avresample --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-gpl --enable-libass --enable-fontconfig --enable-libfreetype --enable-libvidstab --enable-libmp3lame --enable-libopus --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libxcb --enable-libx265 --enable-libxvid --enable-libx264 --enable-nonfree --enable-openssl --enable-libfdk_aac --enable-postproc --enable-small --enable-version3 --enable-libbluray --enable-libzmq --extra-libs=-ldl --prefix=/opt/ffmpeg --enable-libopenjpeg --enable-libkvazaar --enable-libaom --extra-libs=-lpthread --enable-libsrt --extra-cflags=-I/opt/ffmpeg/include --extra-ldflags=-L/opt/ffmpeg/lib
  libavutil      56. 22.100 / 56. 22.100
  libavcodec     58. 35.100 / 58. 35.100
  libavformat    58. 20.100 / 58. 20.100
  libavdevice    58.  5.100 / 58.  5.100
  libavfilter     7. 40.101 /  7. 40.101
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  3.100 /  5.  3.100
  libswresample   3.  3.100 /  3.  3.100
  libpostproc    55.  3.100 / 55.  3.100
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from './Pkh8UtuejGw.mp4':
  Metadata:
    major_brand     : dash
    minor_version   : 0
    compatible_brands: iso6mp41
    creation_time   : 2020-03-12T06:21:25.000000Z
  Duration: 00:03:25.43, start: 0.000000, bitrate: 129 kb/s
    Stream #0:0(eng): Audio: aac (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 6 kb/s (default)
    Metadata:
      creation_time   : 2020-03-12T06:21:25.000000Z
      handler_name    : ISO Media file produced by Google Inc.
Stream mapping:
  Stream #0:0 (aac) -> afade
  areverse -> Stream #0:0 (libvorbis)
Press [q] to stop, [?] for help
Output #0, ogg, to './Pkh8UtuejGw.ogg':
  Metadata:
    major_brand     : dash
    minor_version   : 0
    compatible_brands: iso6mp41
    encoder         : Lavf58.20.100
    Stream #0:0: Audio: vorbis (libvorbis), 44100 Hz, stereo, fltp (default)
    Metadata:
      encoder         : Lavc58.35.100 libvorbis
      major_brand     : dash
      minor_version   : 0
      compatible_brands: iso6mp41
size=    1351kB time=00:00:29.99 bitrate= 368.9kbits/s speed=51.2x
video:0kB audio:1340kB subtitle:0kB other streams:0kB global headers:4kB muxing overhead: 0.831372%
ffmpeg version 4.1.5 Copyright (c) 2000-2020 the FFmpeg developers
  built with gcc 9.2.0 (Alpine 9.2.0)
  configuration: --disable-debug --disable-doc --disable-ffplay --enable-shared --enable-avresample --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-gpl --enable-libass --enable-fontconfig --enable-libfreetype --enable-libvidstab --enable-libmp3lame --enable-libopus --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libxcb --enable-libx265 --enable-libxvid --enable-libx264 --enable-nonfree --enable-openssl --enable-libfdk_aac --enable-postproc --enable-small --enable-version3 --enable-libbluray --enable-libzmq --extra-libs=-ldl --prefix=/opt/ffmpeg --enable-libopenjpeg --enable-libkvazaar --enable-libaom --extra-libs=-lpthread --enable-libsrt --extra-cflags=-I/opt/ffmpeg/include --extra-ldflags=-L/opt/ffmpeg/lib
  libavutil      56. 22.100 / 56. 22.100
  libavcodec     58. 35.100 / 58. 35.100
  libavformat    58. 20.100 / 58. 20.100
  libavdevice    58.  5.100 / 58.  5.100
  libavfilter     7. 40.101 /  7. 40.101
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  3.100 /  5.  3.100
  libswresample   3.  3.100 /  3.  3.100
  libpostproc    55.  3.100 / 55.  3.100
Input #0, ogg, from './Pkh8UtuejGw.ogg':
  Duration: 00:00:30.00, start: 0.000000, bitrate: 368 kb/s
    Stream #0:0: Audio: vorbis, 44100 Hz, stereo, fltp, 499 kb/s
    Metadata:
      ENCODER         : Lavc58.35.100 libvorbis
      MAJOR_BRAND     : dash
      MINOR_VERSION   : 0
      COMPATIBLE_BRANDS: iso6mp41
Stream mapping:
  Stream #0:0 -> #0:0 (vorbis (native) -> aac (native))
Press [q] to stop, [?] for help
Output #0, ipod, to './Pkh8UtuejGw.m4a':
  Metadata:
    encoder         : Lavf58.20.100
    Stream #0:0: Audio: aac (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 128 kb/s
    Metadata:
      COMPATIBLE_BRANDS: iso6mp41
      MAJOR_BRAND     : dash
      MINOR_VERSION   : 0
      encoder         : Lavc58.35.100 aac
size=     479kB time=00:00:30.02 bitrate= 130.8kbits/s speed=55.3x
video:0kB audio:472kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 1.506557%
[aac @ 0x562a55f08640] Qavg: 217.142
```

鈴聲直接輸出在執行 docker 路徑下 `./output` 

```shell
$ ls ./output/
Pkh8UtuejGw.m4r
```

剩下的就是將檔案傳至手機，透過 Garageband 輸出至 iPhone

DONE!

BTW. 有時候再來弄 UI

{{< music url="Pkh8UtuejGw.m4r" name=Wavelength artist=oldmanyoung cover="/images/Wavelength.jpg" >}}
