# Devfest22 Taichung Cloud Workstation Introduction


<!--more-->

{{< slideshare id="253874492" >}}

Slideshare: [DevFest 2022 - Cloud Workstation Introduction TaiChung](https://www.slideshare.net/cagechung/devfest-2022-cloud-workstation-introduction-taichungpdf)

Google announce Gloud Workstation preview version on 2022/10/11. In this year DevFest Taichung 2022, I share Cloud Workstation Introduction. Introduce how to set up Cloud Workstation, use cases and Demo.

### Simple Tips

It's quick easy to create workstation cluster, configuration and workstation. We could leverage [Open VSX Registry](https://open-vsx.org/) to install extension on basic editor [Code-OSS](https://github.com/microsoft/vscode). In reality, we might not found all extension we userd before on Open VSX Registry. Although both of them are open source project, but still quick different a little bit.

Due to Cloud Workation is containerization, so it retains some flexibility in terms of scalability. For instance. Cloud Workstation has alredy integrate with `Cloud Code`, basically it's a plugin for VSCode with few cli tool like `gcloud`, `skaffold` but `helm`. Therefore, we could use `Dockerfile` to build our own image with `helm`. In this way, we could use `helm` in Cloud Workstation container base on our own image.

In ecosytem, Cloud Workstation intergrate with Jetbrain IDEs. We could get same development experience as local workstation base on pre defined Jetrain base container image with [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/) but few knows issues still not support for now.

- Debug on Kubernetes
- Debug a locally running Cloud Run service

Finally, due to Cloud Workstation is preview version now, all of them are subject to change. We could expect more features and better experience in the future. Basically, it's a good start for those developer who want to seprarate development environment from local workstation.
