# Google Announce Application Default Credentials (ADC)


<!--more-->

今天 Google 公佈了 [Application Default Credentials (ADC)](http://googlecloudplatform.blogspot.tw/2015/07/Easier-Auth-for-Google-Cloud-APIs-Introducing-the-Application-Default-Credentials-feature.html)，一個可以讓使用者更方便在 GCP 上去界接其他的需要使用 OAuth 存取的服務如 [Google Cloud Storage](https://cloud-dot-devsite.googleplex.com/storage)、[Google BigQuery](https://cloud-dot-devsite.googleplex.com/bigquery)。這對常常寫 GAE 的我來說又更方便了。

在 GCP 專案建立之後，預設會自動產生 [Service Accounts](https://developers.google.com/accounts/docs/OAuth2ServiceAccount)，
這些內建的 Service Accounts 在進行 Server to Server 的存取時只需要應用程式本身的認証，直接使用 `AppAssertionCredentials` 可以不需透過 `Flow` 來建立 `Credentials`物件


```python
import httplib2
from google.appengine.api import memcache
from apiclient.discovery import build
from oauth2client.appengine import AppAssertionCredentials

credentials = AppAssertionCredentials(scope='https://www.googleapis.com/auth/devstorage.full_control')
http = credentials.authorize(httplib2.Http(memcache))
gcs_service = build('storage', 'v1', http=http, developerKey=DEVELOPER_KEY)
```

我們可以直接使用 `AppAssertionCredentials` 產生的 `credentials` 來建立 `gcs_service` 連線，這段程式在上傳到 GAE 上可以跑的很好，
不過如果你想要在本地測試的時候就需要特別指定 Server Accounts 的 `credentials`

## Service Accounts

Create Service Account

1.	Click **APIs & Auth** > **credential**.
2.	Click **Click new Client ID**.
3.	Choose **Service Account**.
4.	Key type : **P12 Key**
5.	Click **Create Client ID**
6.	**P12.Key** file will be downloaded. You have to convert `p12` to `pem` cause PKCS12 format is not supported by the PyCrypto library.

	```shell
	(openssl pkcs12 -in xxxxx.p12 -nodes -nocerts > privatekey.pem)
	```

在本地測試程式時還需將 `--appidentity_email_address` 及 `--appidentity_private_key_path` 帶到 `dev_appserver.py` 參數中。
如果沒有帶入這二個參數本地測試會得到 `401 Unauthorized` 的錯誤訊息，不過上傳到 GAE 因為會透過應用程式本身的認証所以不會出錯。

```shell
# gae run locally.
dev_appserver.py yaml_or_war_path --appidentity_email_address=<service-account-email> --appidentity_private_key_path=<privatekey.pem-path>
```

## Application Default Credentials (ADC)

而 [Application Default Credentials](https://developers.google.com/identity/protocols/application-default-credentials) 則提供了更簡便的作法

```python
from oauth2client.client import GoogleCredentials
from apiclient.discovery import build

credentials = GoogleCredentials.get_application_default()
gcs_service = build('storage', 'v1', credentials=credentials)
```

Application Default Credentials (ADC) 的方式在本地測試時因為直接使用了 [gcloud auth login](https://cloud.google.com/sdk/gcloud/reference/auth/login) 的 `credentials`，所以在開發上更方便更直覺。

注意事項:

1. [Google APIs Client Library for Python](https://developers.google.com/api-client-library/python/) 中 `1.3` 以上才支援 default credentials
2. `$ gcloud -v` 確認 Google Cloud SDK 版本在 `0.9.51` 以上，Google App Engine SDK 版本在 `1.9.18` 以上

## 參考資料
- [Google Cloud Platform Blog: Easier Auth for Google Cloud APIs. Introducing the Application Default Credentials feature.](http://googlecloudplatform.blogspot.tw/2015/07/Easier-Auth-for-Google-Cloud-APIs-Introducing-the-Application-Default-Credentials-feature.html)
- [Google Application Default Credentials   |   Google Identity Platform   |   Google Developers](https://developers.google.com/identity/protocols/application-default-credentials)
- [Using Google App Engine   |   API Client Library for Python   |   Google Developers](https://developers.google.com/api-client-library/python/guide/google_app_engine)

