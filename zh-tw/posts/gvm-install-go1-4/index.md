# 解決 gvm install go1.4 安裝錯誤 [筆記]


<!--more-->

本來是使用 `brew` 來管理 `golang` 的版本，`brew` 及 `brew cask` 某些程度是很方便，不過也越來越覺得彈性差了點。繼 node 使用 tj 大神的 [tj/n: Node version management](https://github.com/tj/n) 之外，golang 也打算使用 [moovweb/gvm: Go Version Manager](https://github.com/moovweb/gvm) 來管理版本，不過在照著官方步驟時報錯了，順手記錄了一下解決的方式

# brew info go

```bash
$ brew info go
go: stable 1.6 (bottled), HEAD
Go programming environment
https://golang.org
/usr/local/Cellar/go/1.4.2 (4,567 files, 148.2M)
  Built from source
/usr/local/Cellar/go/1.5 (5,328 files, 259.2M)
  Poured from bottle
/usr/local/Cellar/go/1.5.2 (5,336 files, 259.6M)
  Poured from bottle
/usr/local/Cellar/go/1.5.3 (5,337 files, 269.6M)
  Poured from bottle
/usr/local/Cellar/go/1.6 (5,653 files, 472.9M) *
  Poured from bottle
From: https://github.com/Homebrew/homebrew/blob/master/Library/Formula/go.rb
==> Options
--without-cgo
    Build without cgo
--without-godoc
    godoc will not be installed for you
--without-vet
    vet will not be installed for you
--HEAD
    Install HEAD version
==> Caveats
As of go 1.2, a valid GOPATH is required to use the `go get` command:
  https://golang.org/doc/code.html#GOPATH

You may wish to add the GOROOT-based install location to your PATH:
  export PATH=$PATH:/usr/local/opt/go/libexec/bin
```

# fixed gvm install go1.4 報錯

```bash
$ gvm install go1.4
Installing go1.4...
 * Compiling...
ERROR: Failed to compile. Check the logs at /Users/cage/.gvm/logs/go-go1.4-compile.log
ERROR: Failed to use installed version
```

拿錯誤訊息餵狗之後找到 [update readme.md for go1.5 installation by imyelo · Pull Request #176 · moovweb/gvm](https://github.com/moovweb/gvm/pull/176/commits/6d99f7dee6ddd09992620654c2067aaf869b3d1c)

install go1.4 順利完成

```bash
$ gvm install go1.4 -B
Installing go1.4 from binary source
```

指定 golang 版本

```bash
$ gvm use go1.4
Now using version go1.4
```

順利安裝其他版本

```bash
$ gvm install go1.7
Installing go1.7...
 * Compiling...
go1.7 successfully installed!
```

