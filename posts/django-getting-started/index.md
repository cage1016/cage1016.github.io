# django getting started


<!--more-->

Django Quick Getting Started

- pip
- virtualenv
- virtualenvwrapper
- Django
- [twoscoops/django-twoscoops-project](https://github.com/twoscoops/django-twoscoops-project)
- PyCharm

簡單說明如何快速建立一個 Django 專案及搭配 Virtualenv, PyCharm

## Virtualenv

Python 社群擁有非常多的好用的套件，所以 `pip` 的套件管理程式就變的非常的好用。但是這時候就有出現另一個問題。不同專案所需要函數庫可能不盡相同，所以 `Virtualenv` 可以隔離函數庫需求不同的專案，讓它們不會互相影響。

- 在沒有權限的情況下安裝新套件
- 不同專案可以使用不同版本的相同套件
- 套件版本升級時不會影響其他專案

```shell
# install virtualenv (會安裝在全域環境下)
$ pip install virtualenv

# 建立一個 virtualenv, 建立完成後會在 quick_getting_started新增一個 env_QGS 的目錄
$ cd ~/ && mkdir quick_getting_started
$ virtualenv env_QGS

# 執行 Virtualenv, 執行完會再 termianl `$` 號前面出現 virtualenv 的名稱
$ source env_QGS/bin/activateds
(env_QGS) $

# 退出 virtualenv
(env_QGS) $ deactivate
```

**Tips:**

Virtualenv 方便讓我隔離了不同專案下的函式庫，但是我們建立出來的 `env_QGS` (virtualenv 目錄) 會散落在各地，在管理上面還是有一點點麻煩。這時候就有另一個 `virtualenvwrapper` 幫我們解決這一個問題

[virtualenvwrapper](https://virtualenvwrapper.readthedocs.org/en/latest/) 是一個 virtualenv 的強加版，方便我們管理有的 virtualenv

- 將所有的虛擬環境整合在一個目錄下。
- 管理（新增、移除、複製）所有的虛擬環境。
- 可以使用一個命令切換虛擬環境。
- Tab 補全虛擬環境的名字。
- 每個操作都提供允許使用者自訂的 hooks。
- 可撰寫容易分享的 extension plugin 系統。

所以剛剛的操作步驟就可以得到簡化

```shell
# 列出所有的 virtualenv. 所有的 virtualenv 集中管理(~/.virtualenvs/)
$ workon
djago-oscar-api-test
django-oscar-paypal
django-oscar_demo
django-sample-app
github_flask_oauth2
mysite
oauth2app
oscar
....

# 建立 virtualenv env_QGS, 建立完成後會自動幫你啟動(超棒der)
$ mkvirtualenv env_QGS
New python executable in env_QGS/bin/python
Installing setuptools, pip, wheel...done.
virtualenvwrapper.user_scripts creating /Users/cage/.virtualenvs/env_QGS/bin/predeactivate
virtualenvwrapper.user_scripts creating /Users/cage/.virtualenvs/env_QGS/bin/postdeactivate
virtualenvwrapper.user_scripts creating /Users/cage/.virtualenvs/env_QGS/bin/preactivate
virtualenvwrapper.user_scripts creating /Users/cage/.virtualenvs/env_QGS/bin/postactivate
virtualenvwrapper.user_scripts creating /Users/cage/.virtualenvs/env_QGS/bin/get_env_details
(env_QGS) $
```

## Django

> Django是一個開放原始碼的Web應用框架，由Python寫成。採用了MVC的軟體設計模式，即模型M，視圖V和控制器C。它最初是被開發來用於管理勞倫斯出版集團旗下的一些以新聞內容為主的網站的。並於2005年7月在BSD授權條款下釋出。這套框架是以比利時的吉普賽爵士吉他手Django Reinhardt來命名的。[Django - 維基百科，自由的百科全書](https://zh.wikipedia.org/wiki/Django)

```shell
# 使用 mkvirtualenv env_QGS, 列出預設安裝 Python 函式庫
(env_QGS) $ pip list
pip (7.1.2)
setuptools (18.2)
wheel (0.24.0)

# 安裝 Django, 目前最新的版本為 1.8.5
(env_QGS) $ pip django
Collecting django
  Using cached Django-1.8.5-py2.py3-none-any.whl
Installing collected packages: django
Successfully installed django-1.8.5

# 安裝完 Django 後, 會自帶 `django-admin.py` 方便我們進行 Django 專案的建立及其他對應功能
(env_QGS) $ django-admin.py help

Type 'django-admin.py help <subcommand>' for help on a specific subcommand.

Available subcommands:

[django]
    check
    compilemessages
    createcachetable
    dbshell
    diffsettings
    dumpdata
    flush
    inspectdb
    loaddata
    makemessages
    makemigrations
    migrate
    runfcgi
    runserver
    shell
    showmigrations
    sql
    sqlall
    sqlclear
    sqlcustom
    sqldropindexes
    sqlflush
    sqlindexes
    sqlmigrate
    sqlsequencereset
    squashmigrations
    startapp
    startproject
    syncdb
    test
    testserver
    validate

# 建立第一個 Django 專案
(env_QGS) $ django-admin.py startproject my_site

# 列出使用 startproject my_site 後的檔案架構
(env_QGS) $ tree
.
└── my_site                # django project root
    ├── manage.py
    └── my_site            # django app root
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py

# 基本上使用 django-admon 建立完後就可以接直執行
(env_QGS) $ python mysite/manage.py runserver
```

{{<image src="img/django-getting-started-01.png">}}

**Tips:**

使用 `django-admin.py` 可以快速的建立專案，但是在專案進行還是會遇到發開/測試/部署等執行階段，再這些階段會遇到函式庫不同(`requirments.txt`), 設定檔配置也會不同(`settings`)。空專案建立再手動調整的方式就會比較累，所以在專案建立時就引用 `twocoops template` 來加快專案檔案架構的配置

## twoscoops

[Two Scoops Press: Making Python and Django as fun as ice cream](http://twoscoopspress.org/)

{{<image src="img/django-getting-started-02.png">}}

twocoops 是一本書，這本書教你如何最佳實踐 Django 1.8，如上一節提的問題(multiple requirements.txt、settings)等配置方式

```shell
# 我們接著上節的進度 (~/tmp/quick_gettting_started)
(env_QGS) $ ls
my_site # 剛剛建立的 my_site 專案

# 使用 twosoops template
(env_QGS) $ django-admin.py startproject --template=https://github.com/twoscoops/django-twoscoops-project/archive/master.zip --extension=py,rst,html mysite2

# 比較一下 twoscoops 幫我作了什麼
$ tree
.
├── my_site                   # my_site django project root
│   ├── db.sqlite3
│   ├── manage.py
│   └── my_site               # my_site django app
│       ├── __init__.py
│       ├── settings.py
│       ├── urls.py
│       ├── wsgi.py
└── mysite2                   # mysite 2 repo level (如果有使用 git)
    ├── CONTRIBUTORS.txt
    ├── LICENSE.txt
    ├── README.rst
    ├── docs                  # django docs (sphinx)
    │   ├── Makefile
    │   ├── __init__.py
    │   ├── conf.py
    │   ├── deploy.rst
    │   ├── index.rst
    │   ├── install.rst
    │   └── make.bat
    ├── mysite2               # mysite2 django project root (同 my_site django project root level)
    │   ├── manage.py
    │   ├── mysite2           # mysite2 django app root (同 my_site django app root level)
    │   │   ├── __init__.py
    │   │   ├── settings            # multiple settings root
    │   │   │   ├── __init__.py
    │   │   │   ├── base.py
    │   │   │   ├── local.py
    │   │   │   ├── production.py
    │   │   │   └── test.py
    │   │   ├── urls.py
    │   │   └── wsgi.py
    │   ├── static
    │   │   ├── css
    │   │   ├── fonts
    │   │   └── js
    │   └── templates
    │       ├── 404.html
    │       ├── 500.html
    │       └── base.html
    ├── requirements          # multiple requirements root
    │   ├── base.txt
    │   ├── local.txt
    │   ├── production.txt
    │   └── test.txt
    └── requirements.txt
```

{{<image src="img/django-getting-started-03.png">}}

**Tips:**

在執行 mysite2 之前需對建立出來的檔案進行一些修改(因為這個 template 是針對 Django 1.6，我們想要使用 Django 1.8.5 所以必修進行修正)

1. _requirements_
    - `mysite2/requirements/base.txt` : Django==1.6.5 --> Django==1.8.5
    - `mysite2/requirements/local.txt` : django-debug-toolbar==1.2.1 --> django-debug-toolbar==1.4
1. _mysite2/mysite2/mysite2/settings_
    - `base.py`
        1. mark **line 29** `# TEMPLATE_DEBUG = DEBUG`
        2. mark **line 246** `# 'south',` (south support < Django 1.7)
    - `local.py`
        1. mark **line 15** `# TEMPLATE_DEBUG = DEBUG`

```shell
# install package
(env_QGS) $ cd mysite2
(env_QGS) $ pip install -r requirements/local.txt

# database migrate
(env_QGS) $ python mysite2/manage.py migrate
Operations to perform:
  Synchronize unmigrated apps: staticfiles, debug_toolbar, messages
  Apply all migrations: admin, contenttypes, sites, auth, sessions
Synchronizing apps without migrations:
  Creating tables...
    Running deferred SQL...
  Installing custom SQL...
Running migrations:
  Rendering model states... DONE
...

# 執行
(env_QGS) $ python mysite2/manage.py runserver --settings=mysite2.settings.local
```

## Pycharm

Pycharm 是一個很好用的 Python 編輯器，我最喜歡的部份是可以拿來 debug。不過在開發環境上需要設定一下才能配合 `virtualenv` + `twoscoops`

**Project structure 設定 _mysite2(django project level)_ 為source**
{{<image src="img/django-getting-started-04.png">}}

**Project interpreter 加入已經建立好的 virtualenv**
{{<image src="img/django-getting-started-05.png">}}

**設定 Django (Django project, settings, manage.py path)**
{{<image src="img/django-getting-started-06.png">}}

**設定 Run/Debug Configration **
{{<image src="img/django-getting-started-07.png">}}

**Run with fly**
{{<image src="img/django-getting-started-08.png">}}

**Enjoying debug mode**
{{<image src="img/django-getting-started-09.png">}}

## Reference
- [Python 的虛擬環境及多版本開發利器─Virtualenv 與 Pythonbrew - OpenFoundry](http://www.openfoundry.org/tw/tech-column/8516-pythons-virtual-environment-and-multi-version-programming-tools-virtualenv-and-pythonbrew)

