# Golang Base Alfred Code Sign and Notarize


<!--more-->

[Alfred](https://www.alfredapp.com/) is a great tool for improving Mac productivity, and Powerpack allows developers to develop their own workflows to make Alfred even more powerful. One problem when developing with Golang is that the artifact is a compiled binary, and there are security issues with running binary-based Alfred workflows on Mac OS.

## Steps

{{<image src="img/github-action.jpg" alt="Github Action">}}

The entire process of building the Golang-based Alfred workflow package for Github Action consists of several parts

1. Golang `Install`, `Test`, `Build` 等常規操作
   - 1.Install Go
   - 2.Run unit tests
   - 3.update codecov
2. Alfred package packaging and use Gon Code Sign, Notarrize preparation and practical operation
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

Prepare the relevant environment and environment variables and set them to Github (Settings -> Secrets –> Actions –> New repository secret).

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

Using `Gon` to provide code signatures for Golang-based executables of the Alfred workflow package

```bash
# pack alfredworkflow
cd .workflow
plutil -replace version -string "${{ env.tag }}" info.plist
zip -r ../"DevToys-${{ env.tag }}.alfredworkflow" .
cd ..

# zip alfredworkflow as zip archive for notarize
zip -r "DevToys-${{ env.tag }}.alfredworkflow.zip" "DevToys-${{ env.tag }}.alfredworkflow"

```
After the code has been signed, package the complete Alfred workflow as `.alfredworkflow`. Since `.alfredworkflow` is not a file format supported by Notarize, we will need to package it again as a `.zip` file for subsequent work.

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

Because `.zip` packaging does not support `staple` action, in order to avoid subsequent `staple` operation failure, so here first set `staple` to `false`.

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

After the entire Github Action is successfully executed, you will get a `Ready for distribution` log and receive an email notification about it.

## Reference

The full github Action configuration file can be found at https://github.com/cage1016/alfred-devtoys/blob/master/.github/workflows/release.yml
