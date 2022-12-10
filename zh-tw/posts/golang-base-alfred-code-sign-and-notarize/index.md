# Golang Base Alfred Code Sign and Notarize


<!--more-->

[Alfred](https://www.alfredapp.com/) 是一個可以增加 Mac 生產力的好工具，其中的 Powerpack 更是可以讓開發者開發自己的 Workflow，讓 Alfred 運用起來更加強大。在使用 Golang 進行開發的時候會遇到一個問題，就是 Golang 的程式碼最終的產物是編譯後的二進制檔案，在 Mac OS 執行基於二進制檔案的 Alfred workflow 會有安全性的問題出現，自用可能沒有什麼影響，如果需要提供給別人使用的話，就需要正視這個問題，照正規流來走 Code Sign 和 Notarize 來解決這個問題

## Steps

{{<image src="img/github-action.jpg" alt="Github Action">}}

在整個 Github Action 建立 基於 Golang 的 Alfred workflow package 過程中大至包含了幾個部份

1. Golang `Install`, `Test`, `Build` 等常規操作
   - 1.Install Go
   - 2.Run unit tests
   - 3.update codecov
2. Alfred package 打包及使用 Gon Code Sign, Notarrize 的準備工作及實際操作
   - 4.Import Code-Signing Certificates
   - 5.Install gon via HomeBrew for code signing and app notarization
   - 6.Build and pack
   - 7.code sign and notarize
3. Github Action Release
   - 8.upload release assets

```yaml
      - name: Install gon via HomeBrew for code signing and app notarization
        run: |
          brew tap mitchellh/gon
          brew install mitchellh/gon/gon
      - name: Import Code-Signing Certificates
        uses: Apple-Actions/import-codesign-certs@v1
        with:
          # The certificates in a PKCS12 file encoded as a base64 string
          p12-file-base64: ${{ secrets.APPLE_DEVELOPER_CERTIFICATE_P12_BASE64 }}
          # The password used to import the PKCS12 file.
          p12-password: ${{ secrets.APPLE_DEVELOPER_CERTIFICATE_PASSWORD }}
      - name: code sign and notarize
        env:
          AC_USERNAME: ${{ secrets.AC_USERNAME }}
          AC_PASSWORD: ${{ secrets.AC_PASSWORD }}
        run: |
          # gon code sign
          cat <<EOF >> gon.json
          {
              "source" : [".workflow/exe"],
              "bundle_id" : "com.kaichu.devtoys",
              "sign" :{
                  "application_identity" : "Developer ID Application: KAI CHU CHUNG"
              }
          }
          EOF
          gon -log-level=debug -log-json ./gon.json

          # pack alfredworkflow
          cd .workflow
          plutil -replace version -string "${{ env.tag }}" info.plist
          zip -r ../"DevToys-${{ env.tag }}.alfredworkflow" .
          cd ..

          # zip alfredworkflow as zip archive for notarize
          zip -r "DevToys-${{ env.tag }}.alfredworkflow.zip" "DevToys-${{ env.tag }}.alfredworkflow"

          # gon notarize
          cat <<EOF >> notarize.json
          {
              "notarize": [{
                  "path": "${PWD}/DevToys-${{ env.tag }}.alfredworkflow.zip",
                  "bundle_id": "com.kaichu.devtoys",
                  "staple": false
              }]
          }
          EOF
          gon -log-level=debug -log-json ./notarize.json

          echo "artifact=$(echo "DevToys-${{ env.tag }}.alfredworkflow")" >> $GITHUB_ENV
```

## Prerequisite

```yaml
- name: Install gon via HomeBrew for code signing and app notarization
  run: |
    brew tap mitchellh/gon
    brew install mitchellh/gon/gon
- name: Import Code-Signing Certificates
  uses: Apple-Actions/import-codesign-certs@v1
  with:
    # The certificates in a PKCS12 file encoded as a base64 string
    p12-file-base64: ${{ secrets.APPLE_DEVELOPER_CERTIFICATE_P12_BASE64 }}
    # The password used to import the PKCS12 file.
    p12-password: ${{ secrets.APPLE_DEVELOPER_CERTIFICATE_PASSWORD }}
- name: code sign and notarize
  env:
    AC_USERNAME: ${{ secrets.AC_USERNAME }}
    AC_PASSWORD: ${{ secrets.AC_PASSWORD }}
  run: |
    ...
```

準備相關的建置環境及環境變數，並設定至 Github 中 (Settings -> Secrets --> Actions --> New repository secret)

- secrets.APPLE_DEVELOPER_CERTIFICATE_P12_BASE64
- secrets.APPLE_DEVELOPER_CERTIFICATE_PASSWORD
- secrets.AC_USERNAME
- secrets.AC_PASSWORD

__gon.json__

```bash
cat <<EOF >> gon.json
{
    "source" : [".workflow/exe"],
    "bundle_id" : "com.kaichu.devtoys",
    "sign" :{
        "application_identity" : "Developer ID Application: KAI CHU CHUNG"
    }
}
EOF
```

對 Golang Alfred workflow package 的執行檔使用 `Gon` 進行 Code Sign


```bash
# pack alfredworkflow
cd .workflow
plutil -replace version -string "${{ env.tag }}" info.plist
zip -r ../"DevToys-${{ env.tag }}.alfredworkflow" .
cd ..

# zip alfredworkflow as zip archive for notarize
zip -r "DevToys-${{ env.tag }}.alfredworkflow.zip" "DevToys-${{ env.tag }}.alfredworkflow"

```
Code Sign 完後，打包完整的 Alfred workflow package `.alfredworkflow`, 因為 `.alfredworkflow` 並非 Notarize 的支援檔案格式，所以需要再次打包成 `.zip` 檔案，以便後續的 Notarize

__notarize.json__

```bash
cat <<EOF >> notarize.json
{
    "notarize": [{
        "path": "${PWD}/DevToys-${{ env.tag }}.alfredworkflow.zip",
        "bundle_id": "com.kaichu.devtoys",
        "staple": false
    }]
}
EOF
```

因為採用 `.zip` 的 package 不支搜 `staple` 的動作，為了避免後續的 `staple` 操作失敗，所以在這邊先將 `staple` 設定為 `false`

## Result

```json
{
  "logFormatVersion": 1,
  "jobId": "90eac292-fde7-4391-9a8c-1db0210c93aa",
  "status": "Accepted",
  "statusSummary": "Ready for distribution",
  "statusCode": 0,
  "archiveFilename": "DevToys-1.7.1.alfredworkflow.zip",
  "uploadDate": "2022-12-07T15:45:54Z",
  "sha256": "f7de035836559b9f105d5ba96b2b8bcda0d50a0802202b2fdf204e2bcee0d387",
  "ticketContents": [
    {
      "path": "DevToys-1.7.1.alfredworkflow.zip/DevToys-1.7.1.alfredworkflow/exe",
      "digestAlgorithm": "SHA-256",
      "cdhash": "3733bbfae584c96a736b65f589c53edba490148f",
      "arch": "x86_64"
    },
    {
      "path": "DevToys-1.7.1.alfredworkflow.zip/DevToys-1.7.1.alfredworkflow/exe",
      "digestAlgorithm": "SHA-256",
      "cdhash": "111858b884a1ee1e11dca11628a7a3cf09183010",
      "arch": "arm64"
    }
  ],
  "issues": null
}
```

在整個 Github Action 成功執行之後會得到 `Ready for distribution` 的 log 並收到相關的 Email 通知.

## Reference

完整的 github Action 設定檔案可以參考 https://github.com/cage1016/alfred-devtoys/blob/master/.github/workflows/release.yml
