# Wason Language Translator CLI


<!--more-->

Using text translation is a common situation in daily life, such as [Google Translate](https://translate.google.com/?sl=auto&tl=zh-TW&op=docs)、[DeepL Translate](https://www.deepl.com/translator), [Wason Language Translator](https://www.ibm.com/demos/live/watson-language-translator/self-service/home) and other frequently used services.

All of the above services basically support the translation of documents (docx, pptx, pdf, etc.), but the difference is the cost and accuracy of translation.

1. [Cloud Translation | Google Cloud](https://cloud.google.com/translate): Google Cloud Translator provides full document translation, the overall translation effect is better, but there is no free quota, you need to bind a settlement account to use.
1. [Wason Language Translator Demo](https://www.ibm.com/demos/live/watson-language-translator/self-service/home): IBM Cloud also provides file translation. After applying for an account, there is a free plan of 2 MB per file. The overall translation effect is worse than Google Cloud Translate.

This article basically encapsulates Language Translator - IBM Cloud API Docs into cli to facilitate file translation

## Wason Language Translator Lite Plan
{{<image src="img/lite.jpg" alt="lite plan">}}

[Language Translator - IBM Cloud](https://cloud.ibm.com/catalog/services/language-translator)(https://cloud.ibm.com/apidocs/language-translator) The Lite plan gets you started with 1,000,000 characters per month at no cost and includes the default translation models. When you upgrade to a paid plan, you can create custom models.

## Usage

1. Visit 👉 [releases](https://github.com/cage1016/wason-translator-cli/releases) to download latest binary file
1. Visit 👉 [Language Translator - IBM Cloud](https://cloud.ibm.com/catalog/services/language-translator) apply an IBM Cloud account, create Language Translator instance. copy `apiKey` 及 `url`
   
   ```bash
   API_KEY=<replace-your-api-key>
   URL=<replace-url>
   cat <<EOF >> $HOME/.wason-translator-cli.yaml
   api_key: ${API_KEY}
   url: ${URL}
   version: 2018-05-01
   EOF
   ```
1. Translat document by cli and the supported formats are `.doc`, `.docx`, `.ppt`, `.pptx`, `.xls`, `.xlsx`, `.rtf`, `.odt`, `.odp`, `.ods`, `.pdf`, `.htm`, `.html`, `.xml`, `.json`, `.txt`
1. Upload document to translate

   {{< asciinema rows="20" key="472912" start-at="0" loop="true" autoplay="true" speed="2">}}

1. download translted docuemt file

   {{< asciinema rows="20" key="472913" start-at="0" loop="true" autoplay="true" speed="2">}}

1. delete translted docuemt file

   {{< asciinema rows="20" key="472914" start-at="0" loop="true" autoplay="true" speed="2">}}

## Source Code

[cage1016/wason-translator-cli: IBM Cloud Language Translator CLI (document translate)](https://github.com/cage1016/wason-translator-cli)
