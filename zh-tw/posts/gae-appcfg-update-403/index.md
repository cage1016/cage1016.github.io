# gae appcfg update 403


<!--more-->

當 GAE application 本地開發到一個階段時，就會開始想要上傳至 appengine.google.com 進行線上的測試，這時候便會使 command line 工
具來上傳專案

```shell
$ appcfg.py update default/app.yaml ownership/app.yaml
10:46 AM Host: appengine.google.com
10:46 AM Application: <your-application-id>; version: 1
10:46 AM
Starting update of app: <your-application-id>, version: 1
10:46 AM Getting current resource limits.
2016-03-10 10:46:17,616 ERROR appcfg.py:2396 An error occurred processing file '': HTTP Error 403: Forbidden Unexpected HTTP status 403. Aborting.
Error 403: --- begin server output ---
You do not have permission to modify this app (app_id=u's~<your-application-id>').
--- end server output ---
```

重新進行 `gcloud oauth login` & `gcloud project set <your-application-id>`

檢查 `gcloud config`

```shell
$ gcloud config list
Your active configuration is: [default]

[app]
suppress_change_warning = true
[compute]
region = asia-east1
zone = asia-east1-b
[core]
account = <your-account>
disable_usage_reporting = False
project = <your-default-project-id>
```

還是遇到同樣的問題，`HTTP Error 403: Forbidden Unexpected HTTP status 403. Aborting`。

要解決這個問題的方法有二種

1. `--no_cookies` Do not save authentication cookies to local disk.
2. 直接刪除 `.appcfg_cookies` & `.appcfg_oauth2_tokens` 並重新進行 `gcloud oauth login`
  ```shell
  $ la | grep .appcfg_
  -rw-------     1 cage  staff        960 Jul  8  2015 .appcfg_cookies
  -rw-r--r--     1 cage  staff         43 Nov 15 13:01 .appcfg_nag
  -rw-------     1 cage  staff        774 Mar 10 10:33 .appcfg_oauth2_tokens
  ```

