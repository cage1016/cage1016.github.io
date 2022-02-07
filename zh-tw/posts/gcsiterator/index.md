# GCSIterator (Python CSV iterator for Google Cloud Storage) via GAE


<!--more-->

最近的專案常常需要在 [GAE - Python](https://cloud.google.com/appengine/docs/python/) 跟大 CSV (40MB)檔打交道。在 Python 中利用 `csv.reader` & `csv.DictReader`
可以很容易的處理 `csv` 讀取的動作。但是在 GAE 平台上一般 Request 時間只有 **60s**，而 Tasks Request 則有 **10mins** 的限制[3]，而在 GAE 上處理超大檔案的時候除了會遇到
`DeadlineExceededErrors` 的雷也會踩到 `Exceeded soft private memory limit` 的問題(預設 instance 的記憶體只有 **128MB**，在處理大 CSV 檔很容易踩到的雷)

所以在處理大 CSV 檔最好不要一次就把所有的資料讀到記憶體中，而 GAE 上又有檔案存取的限制，所以大部份會搭配 [GCS](https://cloud.google.com/storage/) 一起使用，
把檔案放在 GCS 上，由 GAE 透過 `google-api-python-client` 到 GCS 進行檔案的存取

`google-api-python-client` 中實作了 GCS JSON API 的 chunks 下載(**MediaIoBaseDownload** [4])，在 chunks 下載時就必需另外處理斷行的問題(實作 Python csv.DictReader iterator 內解決斷行問題)


## GCSIterator.py

```python
# GCSIterator.py

import random
import re
import logging
import time

from apiclient.errors import HttpError

DEFAULT_CHUNK_SIZE = 512 * 1024

class GCSIterator(object):
  """
  Reference:
  Parsing Large CSV Blobs on Google App Engine by Daniel Thompson @ d4nt
  http://d4nt.com/parsing-large-csv-blobs-on-google-app-engine

  Implement Google Cloud Storage csv.DictReader Iterator
  """

  def __init__(self, request, progress=0, chunksize=DEFAULT_CHUNK_SIZE):
    self._request = request
    self._uri = request.uri
    self._chunksize = chunksize
    self._progress = progress
    self._init_progress = progress
    self._total_size = None
    self._done = False

    self._last_line = ""
    self._line_num = 0
    self._lines = []
    self._buffer = None
    self._done = False
    self._done_and_last_line = False

    self._bytes_read = 0

    # Stubs for testing.
    self._sleep = time.sleep
    self._rand = random.random

  def __iter__(self):
    return self

  def next(self):
    if (not self._buffer or len(self._lines) == (self._line_num + 1)) and not self._done_and_last_line:
      if self._lines:
        self._last_line = self._lines[self._line_num]

      if not self._done:
        self._buffer, self._done = self.read(3)

      else:
        self._buffer = ''

      self._lines = re.split('\r|\n|\r\n', self._buffer)
      self._line_num = 0

      # Handle special case where our block just happens to end on a new line
      if self._buffer[-1:] == "\n" or self._buffer[-1:] == "\r":
        self._lines.append("")

    if not self._buffer:
      if self._done and not self._last_line:
        raise StopIteration

      else:
        self._done_and_last_line = True

    if self._line_num == 0 and len(self._last_line) > 0:
      # print 'fixing'
      result = self._last_line + self._lines[self._line_num] + "\n"

    else:
      result = self._lines[self._line_num] + "\n"

    # check csv header
    if self._bytes_read == 0 and self._init_progress == 0:
      # if not re.match('email', result.lower().replace('"', '')):
      #   raise ValueError('csv header must contain "email or EMAIL" property.')
      #
      # else:
      result = result.lower()

    self._bytes_read += len(result)
    if not self._done_and_last_line:
      self._line_num += 1

    else:
      self._last_line = ''

    return result

  def read(self, num_retries=0):
    """Get the next chunk of the download.

    Args:
      num_retries: Integer, number of times to retry 500's with randomized
            exponential backoff. If all retries fail, the raised HttpError
            represents the last request. If zero (default), we attempt the
            request only once.

    Returns:
      (status, done): (MediaDownloadStatus, boolean)
         The value of 'done' will be True when the media has been fully
         downloaded.

    Raises:
      apiclient.errors.HttpError if the response was not a 2xx.
      httplib2.HttpLib2Error if a transport error has occured.
    """

    try:

      headers = {
        'range': 'bytes=%d-%d' % (
          self._progress, self._progress + self._chunksize)
      }
      http = self._request.http
      msg = 'read bytes={:d}-{:d}/{}'.format(self._progress,
                                             (self._progress + self._chunksize),
                                             str(self._total_size) if self._total_size else '*')
      logging.info(msg)
      print msg

      for retry_num in xrange(num_retries + 1):
        if retry_num > 0:
          self._sleep(self._rand() * 2 ** retry_num)
          logging.warning(
            'Retry #%d for media download: GET %s, following status: %d'
            % (retry_num, self._uri, resp.status))

        resp, content = http.request(self._uri, headers=headers)
        if resp.status < 500:
          break

      if resp.status in [200, 206]:
        if 'content-location' in resp and resp['content-location'] != self._uri:
          self._uri = resp['content-location']
        self._progress += len(content)

        if 'content-range' in resp:
          content_range = resp['content-range']
          length = content_range.rsplit('/', 1)[1]
          self._total_size = int(length)

        if self._progress == self._total_size:
          self._done = True
        return content, self._done
      else:
        raise HttpError(resp, content, uri=self._uri)

    except Exception as e:
      logging.warning('gcs iterator error manual retry')
      logging.error(e.message)

      self._sleep(self._rand() * 2 ** 2)
      self.read()
```

在 `GCSIterator.py` 中把 GCS JSON API 的 chunks 下載 csv 資料的程式碼植入 iterator 並解決斷行的問題。

## Getting Started

```shell
# Get gcloud
$ curl https://sdk.cloud.google.com | bash

# Get App Engine component
$ gcloud components update app
$ gcloud components update gae-python

# Clone repo from github
$ git clone https://github.com/cage1016/GCSIterator

# Install pip packages
$ sudo pip install -r requirements.txt -t lib
```

Replace your `bucket-name` and `object-name`. You may also modify `chunksize` at line 24 in `main.py` file.

```python
# main.py
...

def get_authenticated_service():
  credentials = GoogleCredentials.get_application_default()
  http = credentials.authorize(httplib2.Http())
  return discovery_build('storage', 'v1', http=http)

gcs_service = get_authenticated_service()
bucket_name = '<your-bucket-name>' # waldo-gcp-file
object_name = '<your-object-name>' # kaichu_1016_00000100.csv

request = gcs_service.objects().get_media(bucket=bucket_name, object=object_name.encode('utf8'))
iterator = GCSIterator(request, chunksize=512)

reader = csv.DictReader(iterator, skipinitialspace=True, delimiter=',')
for row in reader:
  print row
```

Execute `main.py`

```shell
# sample output
$ python main.py
read bytes=0-512/*
{'email': 'kaichu_1016+00000000@gmail.com', 'name': 'cage00000000'}
{'email': 'kaichu_1016+00000001@gmail.com', 'name': 'cage00000001'}
{'email': 'kaichu_1016+00000002@gmail.com', 'name': 'cage00000002'}
{'email': 'kaichu_1016+00000003@gmail.com', 'name': 'cage00000003'}
{'email': 'kaichu_1016+00000004@gmail.com', 'name': 'cage00000004'}
{'email': 'kaichu_1016+00000005@gmail.com', 'name': 'cage00000005'}
{'email': 'kaichu_1016+00000006@gmail.com', 'name': 'cage00000006'}
{'email': 'kaichu_1016+00000007@gmail.com', 'name': 'cage00000007'}
{'email': 'kaichu_1016+00000008@gmail.com', 'name': 'cage00000008'}
{'email': 'kaichu_1016+00000009@gmail.com', 'name': 'cage00000009'}
{'email': 'kaichu_1016+00000010@gmail.com', 'name': 'cage00000010'}
read bytes=513-1025/4411
{'email': 'kaichu_1016+00000011@gmail.com', 'name': 'cage00000011'}
{'email': 'kaichu_1016+00000012@gmail.com', 'name': 'cage00000012'}
{'email': 'kaichu_1016+00000013@gmail.com', 'name': 'cage00000013'}
{'email': 'kaichu_1016+00000014@gmail.com', 'name': 'cage00000014'}
{'email': 'kaichu_1016+00000015@gmail.com', 'name': 'cage00000015'}
{'email': 'kaichu_1016+00000016@gmail.com', 'name': 'cage00000016'}
{'email': 'kaichu_1016+00000017@gmail.com', 'name': 'cage00000017'}
{'email': 'kaichu_1016+00000018@gmail.com', 'name': 'cage00000018'}
{'email': 'kaichu_1016+00000019@gmail.com', 'name': 'cage00000019'}
{'email': 'kaichu_1016+00000020@gmail.com', 'name': 'cage00000020'}
{'email': 'kaichu_1016+00000021@gmail.com', 'name': 'cage00000021'}
{'email': 'kaichu_1016+00000022@gmail.com', 'name': 'cage00000022'}
read bytes=1026-1538/4411
{'email': 'kaichu_1016+00000023@gmail.com', 'name': 'cage00000023'}
{'email': 'kaichu_1016+00000024@gmail.com', 'name': 'cage00000024'}
...
```

## Reference
1. [GAE - Python](https://cloud.google.com/appengine/docs/python/)
2. [13.1. csv — CSV File Reading and Writing — Python 2.7.10 documentation](https://docs.python.org/2/library/csv.html)
3. [Dealing with DeadlineExceededErrors - App Engine — Google Cloud Platform](https://cloud.google.com/appengine/articles/deadlineexceedederrors)
4. [google-api-python-client/http.py at 80da1eff23d7dc02d9f66f82754aa86b55f73be6 · google/google-api-python-client](https://github.com/google/google-api-python-client/blob/80da1eff23d7dc02d9f66f82754aa86b55f73be6/googleapiclient/http.py#L477)
5. [cage1016/GCSIterator](https://github.com/cage1016/GCSIterator)

