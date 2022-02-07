# 如在 Github Pages 建立 Hugo 靜態網站


<!--more-->

### 原由
颱風天那都不能去，只好繼續 Coding 人生、看看 Pocket 未讀的文章。不過也因此發現好幾個不錯的東西

1. [Supercharging the Atom Editor for Go Development · marcio.io](http://marcio.io/2015/07/supercharging-atom-editor-for-go-development/)，最近也開始用 Atom 也開始學習 Golang 語言。這一篇作者說明了他自己的使用經驗。
2. [使用Hugo搭建免费个人Blog · Ulric Qin](http://ulricqin.com/post/how-to-use-hugo/) 這一篇文章看到 Hugo，發現在他的 Blog 也是用 Hugo 架的

在 Ulric Quin 的文章中得知他自己的 Blog 是架在大陸的[GitCafe](https://gitcafe.com/)，Hugo 可以直接發佈到 GitCafe & Github 的 Page **免費**，當然是立刻自己動手玩一玩


在 [Hosting on GitHub Pages](http://gohugo.io/tutorials/github-pages-blog/#configure-git-workflow-the-alternate-way:fcefb200141ace3e7bfd6542457b7a72) 的說明文件中有如何把 Hugo 靜態網站佈署到 Github Pages 中。但因為 [GitHub Pages](https://pages.github.com/) 提供了二種不同形態的頁面 `User or organization site` & `Project Site`。其中個人主頁一個帳號只能有**一個**、而專案頁面則可以**很多個**。

所以在佈署 Hugo 靜態網站也因為對應到不同的型態的 Github Pages 而有所不同。

> 第一種方式是將 Hugo 靜態網站佈署到 Github Page `Project Site` 面頁中. 只需要在 github 上建立一個 repo，但是利用 git 中 `subtree` 的概念將 `public` 資料夾連結到 `gh-pages` 的分支上，git 操作過程較為繁鎖

Url 上的差異
```shell
# project site url
http://github.com/<your-github-account>/<hugo-project-name>

# github pages host url
http://<your-github-account>.github.io
```

#### Project Site

#### Step1 - 安裝 Hugo 並建立新專案
在[安裝](https://github.com/spf13/hugo/releases)(詳細步驟請看這)好 Hugo 後，直接建立新的 Hugo 專案

```shell
# 建立 Hugo 新專案，-f 是指定 yaml 格式，預設為 toml:frontmatter format
$ hugo new site hugo_blog -f yaml

# change directory
$ cd hugo_blog

# git initialized
$ git init
$ echo .DS_Store >> .gitignore

# add git remote repo
$ git remote add origin git@github.com:<your-github-account>/hugo_blog.git
```

檢視新專案資料架構

```shell
# <project-name> file structure
├── archetypes
├── config.toml
├── content
├── data
├── layouts
└── static
```

#### Step2 - [安裝新的 Themes](http://gohugo.io/themes/installing/)
> [Hugo Themes repository](https://github.com/spf13/hugoThemes)

本來安裝 themes 可直接新建 themes 資料夾並使用 `git clone <themes-url>`，不過此方法在後面 push 到 Github Pages 出，Github 會回報

>The page build failed with the following error: The submodule `themes/hyde` was not properly initialized with a `.gitmodules` file. For more information, see >https://help.github.com/articles/page-build-failed-missing-submodule.
>If you have any questions you can contact us by replying to this email.

所以這邊直接使用 git `submodule` 的方式來安裝 themes
```shell
# add hugo themes to project as submodule
# git submodule add <repository> [<path>]
$ git submodule add https://github.com/spf13/hyde themes/hyde
```

#### Step3 - 編輯專案設定檔

```bash
baseurl : "http://<your-github-account>.github.com/hugo_blog"
languageCode : "en-us"
title : "My New Hugo Site"

# 新增 theme 的名稱
theme : 'hyde'
...
```

#### Step4 - 新增新的文章
在 `content/posts` 建立 `first-post.md`

```shell
# 會在專案 content/posts 下產生 first-post.md 檔案
# -f 使用 yaml 檔案格式
$ hugo new posts/first-post.md -f yaml
```

編輯 `first-post.md`

```markdown
---
date: 2015-07-16T23:01:57+08:00
title: first post
---

This is my first post.
```

#### Step5 - 預覽
此時就可以在 Local 執行， [http://127.0.0.1:1313/hugo_blog/](http://127.0.0.1:1313/hugo_blog/)

```shell
# -w watch filesystem for changes and recreate as needed
# -D include content marked as draft
# Press Ctrl+C to stop
$ hugo server -w
```
應該可以看到下面擷圖的樣式
{{<image src="img/first-post-screenshot.jpg">}}

#### Step6 - 發佈 Hugo 靜態網站至 Github Pages

接下來的動作是一連串的 git 操作，把 Hugo 產生的 `public` 資料夾推送至 Github Pages

```shell
# remove, public folder will created later
$ rm -rf public

$ git add .
$ git commit -m 'hugo project init'

# push
$ git push -u origin master

# Create a new orphand branch (no commit history) named gh-pages
$ git checkout --orphan gh-pages

# Unstage all files
# -rf themes/hyde
$ git rm -rf --cached $(git ls-files)

# Add and commit that file
$ git add .
$ git commit -m "INIT: initial commit on gh-pages branch"

# Push to remote gh-pages branch
$ git push origin gh-pages

# Return to master branch
$ git checkout master

# Remove the public folder to make room for the gh-pages subtree
$ rm -rf public

# Add the gh-pages branch of the repository. It will look like a folder named public
$ git subtree add --prefix=public git@github.com:<your-github-account>/hugo_blog.git gh-pages --squash

# Pull down the file we just committed. This helps avoid merge conflicts
$ git subtree pull --prefix=public git@github.com:<your-github-account>/hugo_blog.git gh-pages

# Run hugo. Generated site will be placed in public directory (or omit -t ThemeName if you're not using a theme)
$ hugo

# Add everything
$ git add -A

# Commit and push to master
$ git commit -m "Updating site" && git push origin master

# Push the public subtree to the gh-pages branch
$ git subtree push --prefix=public git@github.com:<your-github-account>/hugo_blog.git gh-pages
```

這時候，訪問 [http://your-github-account.github.com/hugo_blog](http://<your-github-account>.github.com/hugo_blog) 應該就可以正常運作

之後如果有任何修改，只有執行最後的3個步驟即可或是將最後的步驟寫成 `deploy.sh`

```shell
# deploy.sh
echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo

# Add changes to git.
git add -A

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master
git subtree push --prefix=public git@github.com:<your-github-account>/hugoblog.git gh-pages
```

### Organization site

Github Pages 除了提供專案主頁(可以多個)之外，也提供了個人主頁(每一個 github 帳號只有一個)的方式。將 Hugo 靜態網站發佈到個人主頁的方式比發佈到專案主頁簡單

在個人主頁中

- Github repo 必需取口 <your-github-account>.github.io
- master 分支中的內容會被 Build 及發佈到你的 Github Page中 (專案主頁是利用 gh-pages 分支，這點不太一樣)

#### Step1 - 建立 Hugo repos

- 建立 `<github-project>-hugo` repo (用來 host Hugo 的內容)
- 建立 `<your-github-account>.github.io` repo (Hugo public 中靜態網頁的內容)

#### Step2 - 建立 Hugo 新專案

```shell
$ hugo new site <github-project>-hugo -f yaml

# change directory
$ cd <github-project>-hugo

# git initialized
$ git init
$ echo .DS_Store >> .gitignore

# add git remote repo
$ git remote add origin git@github.com:<your-github-account>/<github-project>-hugo.git
```

#### Step3 - 安裝 Themes

```shell
$ git submodule add https://github.com/spf13/hyde themes/hyde
```

#### Step4 - 編輯專案設定檔

```shell
baseurl: 'http://<your-github-account>.github.com/'
languageCode: 'en-us'
title: 'My New Hugo Site'

theme: 'hyde'
...
```

#### Step5 - 新增文章

```shell
$ hugo new posts/first-post.md -f yaml
```

```markdown
---
date: 2015-07-19T17:32:25+08:00
title: first post
---

This is my first hugo post
```

#### Step6 - 預覽
此時就可以在 Local 執行， [http://127.0.0.1:1313/hugo_blog/](http://127.0.0.1:1313/hugo_blog/)

```shell
# -w watch filesystem for changes and recreate as needed
# -D include content marked as draft
# Press Ctrl+C to stop
$ hugo server -w
```

#### Step7 - 移除 public

```shell
# it will created by `hugo` command after we executed `deploy.sh`
$ rm -rf public
```

## Step8 - 新增 <your-github-account>.github.io public as submodule

```shell
$ git submodule add git@github.com:<your-github-account>/<your-github-account>.github.io.git public
```

#### Step8 - 發佈

```shell
#deploy.sh
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace by `hugo -t <yourtheme>`

# Go To Public folder
cd public
# Add changes to git.
git add -A

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back
cd ..
```

執行發佈shell後，內容會被推送到 `<your-github-account>-hugo`，而 `public` 會被推送到 `<your-github-account>.github.io`

```shell
$ deploy.sh 'your commit message'
```

待 Github Page 編譯發佈後，訪問 [http://your-github-account.github.io](http://your-github-account.github.io) 就會看到結果!

### 參考資料
- [快速搭建gohugo博客 · Mac Zealot - A.C Che](http://imcc.mobi/posts/How-to-build-gohugo-on-github/)
-	[Hosting on GitHub Pages](http://gohugo.io/tutorials/github-pages-blog/#configure-git-workflow-the-alternate-way:fcefb200141ace3e7bfd6542457b7a72)
- [GitHub Pages + GoDaddy — Medium](https://medium.com/@LovettLovett/github-pages-godaddy-f0318c2f25a)

