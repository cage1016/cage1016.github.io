# Pack 0.19.0 Solved Invalid Cross Device Link at Google Cloud Build


<!--more-->

在前幾篇介紹 Pack 時就有提供 Google Cloud Build 使用 Pack 來建構 Container image 時因為 Pack 的 Cached 預設目錄設置的方式導至 Google Cloud Build 過程中遇到 `Google Cloud Build` 的錯誤訊息，當時就有回報這個問題

- [cloudb build `gcr.io/k8s-skaffold/pack` got invalid cross-device link ERROR with custom buildpack · Issue #135 · GoogleCloudPlatform/buildpacks](https://github.com/GoogleCloudPlatform/buildpacks/issues/135)
- [Create temporary registry cache alongside intended destination by briandealwis · Pull Request #1183 · buildpacks/pack](https://github.com/buildpacks/pack/pull/1183)

雖然是使用 [GoogleCloudPlatform/buildpacks](https://github.com/GoogleCloudPlatform/buildpacks) 中遇到的問題，不過實際上底層也是去呼叫 [buildpacks/pack](https://github.com/buildpacks/pack)，所以這個 `invalid cross-device link` 只能等到 pack 發佈 `0.19.0` 才能解決(當時的 Roadmap 就排定這個版本會包含)

終於 [Release pack v0.19.0 · buildpacks/pack](https://github.com/buildpacks/pack/releases/tag/v0.19.0) 在 19 天前發佈了，我們在 Google Cloud Build 上使用 Pack 來建構 Container image 就不需要使用 workaround 的方式，也可以享受 Google Cloud Build parallel 減少整體執行時間的好處

### Before

```yaml
steps:
  - name: gcr.io/k8s-skaffold/pack
    entrypoint: sh
    env:
      - TMPDIR=/builder/home/.pack
    args:
      - -exc
      - |
        pack build --builder=gcr.io/buildpacks/builder:v1 --env=GOOGLE_BUILDABLE=cmd/streamsvc/main.go --descriptor=project-asset.toml 100
        pack build --builder=gcr.io/buildpacks/builder:v1 --path=internal/app/apitest 101

  - name: gcr.io/k8s-skaffold/pack
    entrypoint: sh
    args:
      - -exc
      - |
        pack build --builder=gcr.io/buildpacks/builder:v1 --env=GOOGLE_BUILDABLE=cmd/ws/main.go --descriptor=project-default.toml 200
```

### After

```yaml
steps:
  - name: gcr.io/retailbase-dev/pack:v0.19.0
    entrypoint: sh
    args:
      - -exc
      - |
        pack build --builder=gcr.io/buildpacks/builder:v1 --env=GOOGLE_BUILDABLE=cmd/streamsvc/main.go --descriptor=project-asset.toml 100
        pack build --builder=gcr.io/buildpacks/builder:v1 --path=internal/app/apitest 101

  - name: gcr.io/retailbase-dev/pack:v0.19.0
    entrypoint: sh
    args:
      - -exc
      - |
        pack build --builder=gcr.io/buildpacks/builder:v1 --env=GOOGLE_BUILDABLE=cmd/ws/main.go --descriptor=project-default.toml 200
```

Pack `0.19.0` 就不需要特別指定 `TMPDIR=/builder/home/.pack` 參數，這樣我們可以依照需求來配置適合的步驟

{{< admonition warning "gcr.io/k8s-skaffold/pack" true >}}
Skaffold team 維護的 [gcr.io/k8s-skaffold/pack](https://console.cloud.google.com/gcr/images/k8s-skaffold/GLOBAL/pack?gcrImageListsize=30) 版本還沒有更新至 v0.19.0。[gcr.io/k8s-skaffold/pack 0.19.0 support · Issue #6028 · GoogleContainerTools/skaffold](https://github.com/GoogleContainerTools/skaffold/issues/6028)，在官方還沒有更新之前大家可以[參考](https://github.com/GoogleContainerTools/skaffold/blob/master/deploy/buildpacks/publish.sh) 官方的方式自行打包
{{< /admonition >}}
