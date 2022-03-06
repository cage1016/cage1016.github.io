# Wason Language Translator CLI


<!--more-->

使用文字翻譯已經日常生活當中經常會遇到的情境，如 [Google Translate](https://translate.google.com/?sl=auto&tl=zh-TW&op=docs)、[DeepL Translate](https://www.deepl.com/translator), [Wason Language Translator](https://www.ibm.com/demos/live/watson-language-translator/self-service/home) 等經常使用的服務。

最近在工作環境中製作簡報會有遇到在地中文化的需求，上述的服務基本上也都支援了文件的翻譯功能 (docx, pptx, pdf 等等格式)，其中不太一樣的地方就是使用的費用還有翻譯的精準度。

1. [Cloud Translation | Google Cloud](https://cloud.google.com/translate): Google Cloud Translation 有提供整供文件翻譯，整體翻譯較佳，不過沒有任何免費的額度，使用上需要綁定計費帳戶
1. [Wason Language Translator Demo](https://www.ibm.com/demos/live/watson-language-translator/self-service/home): IBM Cloud 也有提供文件翻譯，申請帳號之後會有 2 MB一個檔案的免費方案，整體翻譯效果較 Google Cloud Translation 差一些，勘用。

本篇文章基本上就在將 [Language Translator - IBM Cloud API Docs](https://cloud.ibm.com/apidocs/language-translator) 封裝成 cli 來方便進行文件翻譯 

## Wason Language Translator Lite Plan
{{<image src="img/lite.jpg" alt="lite plan">}}

[Language Translator - IBM Cloud](https://cloud.ibm.com/catalog/services/language-translator)(https://cloud.ibm.com/apidocs/language-translator) Lite 計劃每月免費提供 1,000,000 個字符，包括預設翻譯模型。 當您升級到付費計劃時，您可以創建自定義模型。

## 使用方法

1. 點擊 👉 [releases](https://github.com/cage1016/wason-translator-cli/releases) 下載最新的執行檔
1. 點擊 👉 [Language Translator - IBM Cloud](https://cloud.ibm.com/catalog/services/language-translator) 申請一個 IBM Cloud 帳戶，並建立 Language Translator 應用實例，複製 `apiKey` 及 `url`
   
   ```bash
   API_KEY=<replace-your-api-key>
   URL=<replace-url>
   cat <<EOF >> $HOME/.wason-translator-cli.yaml
   api_key: ${API_KEY}
   url: ${URL}
   version: 2018-05-01
   EOF
   ```
1. 使用 cli 進行文件翻譯，支援的格式有  `.doc`, `.docx`, `.ppt`, `.pptx`, `.xls`, `.xlsx`, `.rtf`, `.odt`, `.odp`, `.ods`, `.pdf`, `.htm`, `.html`, `.xml`, `.json`, `.txt`
1. 上傳文件檔案

   {{< asciinema rows="20" key="472912" start-at="0" loop="true" autoplay="true" speed="2">}}

1. 下載翻譯好的文件

   {{< asciinema rows="20" key="472913" start-at="0" loop="true" autoplay="true" speed="2">}}

1. 刪除文件

   {{< asciinema rows="20" key="472914" start-at="0" loop="true" autoplay="true" speed="2">}}

## 程式原始碼

[cage1016/wason-translator-cli: IBM Cloud Language Translator CLI (document translate)](https://github.com/cage1016/wason-translator-cli)
