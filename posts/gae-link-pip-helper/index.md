# gae link pip helper


<!--more-->

Python 的社群非常活躍有非常多好用的套件可以使用，也有 `pip` 的套件管理程式來讓開發者管理套件升級、版控等問題


## virtualenv

一般在發者 Python 相關的專案會引入 `virtualenv` 的概念來區隔多專案發開套件的相依問題。如果沒有導入 virtualenv 時 `pip install` 會直接把套件安裝在全域環境，專案也許也引用某些套件特定的版次，此種方式會造成開發上的擾困。基本上都會建議使用 `virtualenv`

## virtualenvwrapper

`virtualenvwrapper` 是 `virtualenv` 的加強版。它會把套件安裝的目錄統一收到 `${HOME}/.virtualenvs` 下集中管理，也提供更方便建立、移除、切換 virtualenv 的指令

## GAE

在開發 Google App Engine - Python 的時候，在做用第三方套件的時候會有一些限制 ([Libraries in Python 2.7 - Python — Google Cloud Platform](https://goo.gl/GV1oxu))，Google App Engine 的執行環境中有支援了一些第三方套件，引用的方式是直接在專案的 `app.yaml` 中指定即可

```bash
libraries:
- name: PIL
  version: "1.1.7"
- name: webob
  version: "1.1.1"
- name: jinja2
  version: latest  
```

而 Google App Engine 的執行環境中沒有支援的第三方套件則需要將引用到的第三方套件程式源始碼以`檔案`方式一同上傳到 Google App Engine 平台才可以正常執行。一般進行 `pip` 套件安裝時會裝在當前的 `virtualenv` 或 全域的環境變數中，但是搭配 GAE 指則需要使用 `pip install -r requirements.txt -t lib` 套件裝在當然目錄 `lib` 並在 `appengine_config.py` 中將 `lib` 目錄加到 `sys` path 中

```python
import os, sys
sys.path.insert(0, os.path.join(os.path.dirname(__file__), 'lib'))
```

## [How to use Google App Engine with virtualenv | Vita Smid ~ Zephyrus | Mathematics, philosophy, code, travel and everything in between.](http://ze.phyr.us/appengine-virtualenv/)

文章作者開發了一個 `linkenv` 的套件可以利用指令快速用 symbol link 建立 `virtualenv` 到本地的資料夾。

## link pip helper Shell

```bash
$ virtualenv env
$ source bin/env/activate
(env)$ pip install djangoappengine six ...
(env)$ pip install -r requirements.txt ...
```

因為 [How to use Google App Engine with virtualenv | Vita Smid ~ Zephyrus | Mathematics, philosophy, code, travel and everything in between.](http://ze.phyr.us/appengine-virtualenv/) 文章提及的方式只有使用到 `virtualenv` 並沒有使用 `virtualenvwrapper`，所以每次新專案步驟重置就會花比較多的時間。

__link pip.sh__

```bash
#!/usr/bin/env bash
# [How to use Google App Engine with virtualenv | Vita Smid ~ Zephyrus |
#   Mathematics, philosophy, code, travel and everything in between.](http://ze.phyr.us/appengine-virtualenv/)
# virtualenv pip package link helper
# rm -rf gaenv
# linkenv $VIRTUAL_ENV/lib/python2.7/site-packages gaenv

# Default linker foler
DEFAULT_LINK_FOLDER=gaenv

if [[ $# -eq 0 ]]
then
  LIB=$DEFAULT_LINK_FOLDER
else
  LIB=$1
fi

# relink
rm -rf $DEFAULT_LINK_FOLDER
rm -rf $LIB
linkenv $VIRTUAL_ENV/lib/python2.7/site-packages $LIB
```

為了減少 Google App Engine 專案建立的時間，將上述的 shell 加入 `.bash_profile` 搭配 `virtualenv` `virtualenvwrapper` `requirements.txt` 就可以簡易的建置開發環.

## Quick tutorial

Github repo: [cage1016/link-pip-helper](https://github.com/cage1016/link-pip-helper)

```bash
# git clone repo
$ git clone git@github.com:cage1016/link-pip-helper.git
Cloning into 'link-pip-helper'...
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 7 (delta 0), reused 7 (delta 0), pack-reused 0
Receiving objects: 100% (7/7), done.
Checking connectivity... done.

# switch to link-pip-helper
$ cd link-pip-helper/

# 利用 virtualenvwrapper 建立 virtualenv
$ mkvirtualenv link-pip-helper-tutorial
New python executable in /Users/{HOME}/.virtualenvs/link-pip-helper-tutorial/bin/python2.7
Also creating executable in /Users/{HOME}/.virtualenvs/link-pip-helper-tutorial/bin/python
Please make sure you remove any previous custom paths from your /Users/{HOME}/.pydistutils.cfg file.
Installing setuptools, pip, wheel...done.
virtualenvwrapper.user_scripts creating /Users/{HOME}/.virtualenvs/link-pip-helper-tutorial/bin/predeactivate
virtualenvwrapper.user_scripts creating /Users/{HOME}/.virtualenvs/link-pip-helper-tutorial/bin/postdeactivate
virtualenvwrapper.user_scripts creating /Users/{HOME}/.virtualenvs/link-pip-helper-tutorial/bin/preactivate
virtualenvwrapper.user_scripts creating /Users/{HOME}/.virtualenvs/link-pip-helper-tutorial/bin/postactivate
virtualenvwrapper.user_scripts creating /Users/{HOME}/.virtualenvs/link-pip-helper-tutorial/bin/get_env_details

# pip 安裝相依套件
$ pip install -r requirements.txt
Collecting google-api-python-client==1.4.0 (from -r requirements.txt (line 1))
....

# 執行 shell (你可以將 link pip.sh 內容 export 成任何 function 名)
$ link_pip lib

# 本地執行 Google App Engine
$ dev_appserver.py .
```

