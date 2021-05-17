# 筆記 - django oscar i18n


<!--more-->

[django-oscar](https://github.com/django-oscar/django-oscar) 是一個滿完整的開源的 EC 專案。Oscar 內建 i18n 是使用 [transifex](https://www.transifex.com/codeinthehole/django-oscar/)，依照 django-oscar 的說明，當 django-oscar 進行 master 的發佈時才會去更新相關的翻譯檔案，如果 i18n 的更新速度沒有辦法符合自己的需求有二種方式，1)加入 transifex 專案幫忙翻譯，只是 django-oscar 的更新速度比較慢 2)使用本地的檔案


目前 django-oscar 支援的 i18n

```shell
├── am_ET
├── ar
├── ar_SA
├── bg_BG
├── bn
├── bn_BD
├── ca
├── cmn
├── cs
├── cs_CZ
├── da
├── de
├── el_GR
├── en
├── en_US
├── es
├── es_AR
├── es_CL
├── et
├── eu
├── fa
├── fa_IR
├── fi
├── fr
├── he
├── hi
├── hi_IN
├── id
├── it
├── ja
├── ka_GE
├── ko
├── lt
├── nb_NO
├── nl
├── pl
├── pt_BR
├── pt_PT
├── ro_RO
├── ru
├── ru_RU
├── sk
├── sk_SK
├── sv
├── sv_SE
├── th_TH
├── tr
├── tr_TR
├── uk
├── uk_UA
├── vi
├── zh
├── zh-Hant
├── zh_CN
├── zh_HK
├── zh_TW
└── zh_TW.Big5
```

django-oscar 使用 i18n 的設定，如果需要使用本地的 Locale 檔案，需要另外指定 `LOCALE_PATHS`

__settings.py__

```python
########## GENERAL CONFIGURATION
# See: https://docs.djangoproject.com/en/dev/ref/settings/#time-zone
TIME_ZONE = 'Asia/Taipei'

# See: https://docs.djangoproject.com/en/dev/ref/settings/#language-code
LANGUAGE_CODE = 'en-us'

# Includes all languages that have >50% coverage in Transifex
# Taken from Django's default setting for LANGUAGES
gettext_noop = lambda s: s
LANGUAGES = (
  ('en-us', gettext_noop('English (United States)')),
  ('zh-cn', gettext_noop('Simplified Chinese')),
  ('zh-tw', gettext_noop('Chinese (Taiwan)')),
)

# Locale Path
LOCALE_PATHS = (
  normpath(join(DJANGO_ROOT, 'locale')),
)

# See: https://docs.djangoproject.com/en/dev/ref/settings/#site-id
SITE_ID = 1

# See: https://docs.djangoproject.com/en/dev/ref/settings/#use-i18n
USE_I18N = True

# See: https://docs.djangoproject.com/en/dev/ref/settings/#use-l10n
USE_L10N = True

# See: https://docs.djangoproject.com/en/dev/ref/settings/#use-tz
USE_TZ = True
########## END GENERAL CONFIGURATION
```

在建立自己的 django-oscar EC 商城時是使用 `pip install django-oscar` 來安裝，所以要重製所有 django-oscar 的翻譯 message 時就必需建立 `$PATH_TO_OSCAR` 的 symlink, 指定 symlink 之後執行 `python ./manage.py makemessages` 才會到 django-oscar env 的目錄把相關的 message 字串建立成 `*.go` 檔

## Simple go through

使用 [generator-django-oscar-app](https://github.com/cage1016/generator-django-oscar-app) 快速建立專案。依照連結的操作方式完成後(應該已經建立專案目錄，virtualenv)

切換至工作目錄

```shell
(env) cd {project_path}/sandbox
```

建立 `i18n` 及 `locale` 目錄

```shell
(env) mkdir i18n locale
```

link virtualenv django-oscar source

```shell
(env) ln -s ~/.virtualenvs/{your-ven-name}/lib/python2.7/site-packages/oscar i18n/oscar
```

使用 `makemessage` 建立 `zh_TW` locale 檔

```shell
# django-oscar locale: sandbox/i18n/oscar/locale
# project locale: sandbox/locale
(env) python manage.py makemessages --symlinks --locale=zh_TW
```

編譯 messages

```shell
(env) python manage.py compilemessages
```

