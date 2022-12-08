# Devfest22 Taipei Skaffold2 Deep Dive


<!--more-->

{{< slideshare id="254778592" >}}

Slideshare: [DevFest 2022 - Skaffold 2 Deep Dive Taipei.pdf](https://www.slideshare.net/cagechung/devfest-2022-skaffold-2-deep-dive-taipeipdf)

A brief review of what i have shared at the Devfest from 2018 to 2021. 

- [Devfest19 Build Go Kit Microservices at Kubernetes With Ease](https://kaichu.io/zh-tw/posts/devfest19-build-go-kit-microservices-at-kubernetes-with-ease/)
- [Devfest20 How to Debug Microservices on Kubernetes as a Pros](https://kaichu.io/zh-tw/posts/devfest20-how-to-debug-microservices-on-kubernetes-as-a-pros/)
- [Coscup X Ruby Conf Tw 2021 Google Cloud Buildpacks](https://kaichu.io/zh-tw/posts/coscup-x-ruby-conf-tw-2021-google-cloud-buildpacks/) - COSCUP
- [Devfest21 Taipei Artifact Registry Introduction](https://kaichu.io/zh-tw/posts/devfest21-taipei-artifact-registry-introduction/)
- [Devfest22 Taichung Cloud Workstation Introduction](https://kaichu.io/zh-tw/posts/devfest22-taichung-cloud-workstation-introduction/)


At 2022 Taipei GDG Devfest session, i talk about the new features of Skaffold 2, a very developer-friendly tool that will help you if you are a Kubernetes application developer.

Although this was said to be an deep dive session on Skaffold 2, we surveyed the audience about their use of Skaffold. Most of the people who attended the conference had not used it beyond that. The questions asked focused on the underlying technology used by Skaffold 2, such as the way containers are built via buildpack rather than `Dockerfile', applicability，barriers to entry and the role of Skaffold in CI/CD, etc. These are actually good questions to ask.

{{<image src="img/005.jpg" alt="Skaffold">}}

Skaffold is a developer-centric tool designed to enable developers to quickly develop, test, and deploy applications on Kubernetes. The pipeline phase is flexible and can be chosen according to your preferences and scenarios. For example, I personally prefer to use buildpack instead of `Dockerfile`, but what if I'm dealing with a non-mainstream scenario? In that case, you can still choose to use `Dockerfile` to build containers with full autonomy and control.

## Timeline
{{<image src="img/placeholder.jpg" alt="timeline">}}

From this picture, we can see that Skaffold is actually developing very fast

1. `v0.1.0` was released on 2018/3/6, and after 46 releases, it was released as `v1.1.0` on 2019/12/21.
1. `v1.1.0` was released, and after 72 iterations in 3 years, it was released on 2022/10/21 as `v.2.0.0`.

## Highlights & New Features
- 💻 Support for deploying to ARM, X86 or Multi-Arch K8s clusters from your x86 or ARM machine
- 👟 New Cloud Run Deployer brings the power of Skaffold to Google Clouds serverless container runtime
- 📜 Skaffold render phase has been split from deploy phase providing increased granularity of control for GitOps workflows
- 🚦New Skaffold verify phase enables improved testing capabilities making Skaffold even better as a CI/CD tool
- ⚙ Tighter integration with kpt lets you more dynamically manage large amounts of configuration and keep it in sync

[Release v2.0.0 Release · GoogleContainerTools/skaffold](https://github.com/GoogleContainerTools/skaffold/releases/tag/v2.0.0)

Skaffold V2 extends the platforms and architectures supported by Skaffold, introduces Cloud Run as a supported deployer, and now supports building and deploying from ARM and x86 architectures. Skaffold V2 also provides enhanced support for CI/CD and GitOps workflows, introduces Skaffold rendering, validation and kpt integration. Most importantly, all existing Skaffold configurations are fully compatible with Skaffold V2, so upgrading from V1 is as easy as running a Skaffold fix.

## Skaffold Pipeline Stages

{{<image src="img/021.jpg" alt="Architecture">}}
{{<image src="img/024.jpg" alt="Skaffold Pipeline Stages">}}
{{<image src="img/022.jpg" alt="Architecture - Cloud Build / Helm">}}

Skaffold Architecture/Pipeline Stages is the core of Skaffold, which provides a Pipeline architecture around `Build`, `Test`, `Tag`, and `Deploy`. If you are familiar with the usage and application of each Pipeline Stage, you will be more familiar with the use of Skaffold and you will be able to use Skaffold more freely to develop, test and deploy applications.

## Tips & Tricks

{{<image src="img/007.jpg" alt="Install">}}

Skaffold is also integrated with existing IDEs (VSCode / IntellJ) / Google Cloud Shell / Cloud Workstation for quick and easy use.

{{< admonition warning "Code Code" true >}}
Cloud Code (**v1.20.4**, Nov 2022) not support Skaffold 2 yet, current build-in skaffold version is **v1.39.3**
{{< /admonition >}}
