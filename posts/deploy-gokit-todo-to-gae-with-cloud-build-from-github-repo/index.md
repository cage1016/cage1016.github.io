# Deploy Gokit Todo to GAE With Cloud Build From Github Repo


<!--more-->

## gokit-todo 

https://github.com/cage1016/gokit-todo

>todomvc full stack demo project. react + backend API by gokit microservice toolkit. include unit test, integration test, e2e test, github action ci

https://github.com/cage1016/gokit-todo-frontend

> gokit todo frotnend: React todomvc

{{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/demo.gif" alt="gokit-todo demo">}}

`gokit-doto` (Microservices backend API with Postgres database) and `gokit-todo-frontend` (React todomvc) are demo applications developed for Kubernetes with Ingress (Istio/Nginx-ingress). You can also quickly start an entire application with a database via docker-compose. Feel free to try it out by following the guide in the repo.

## migrate gokit-todo gokit-todo-frontent to Google App Engine with Cloud SQL

Before we migrated our Kubernetes application to Google App Engine. We first need to have some understanding of Google App Engine. Make sure our ideas work.

1. Google App Engine is a Pass (Platform as a Service) service, which means we only need to focus on application development. So we can reuse gokit-todo, gokit-todo-frontend source code and deploy to Google App Engine through `app.yaml` configuration ✅
1. Google App Engine support standard-runtime (`Python`, `Java`, Node.`js`, `PHP`, `Ruby`, `Go`) and flexible-runtime (`Go`, `Java 8`, `PHP 5/7`, `Python 2.7/3.6`, `.NET`, `Node.js`, `Ruby`, `Custom runtime`)
   - Choose standard runtime Go 1.16 for gokit-todo as it is a pure backend API microservice ✅
   - Chosee standard-runtime Node.js 16 for gokit-todo-frontentd as it is React frontend application ✅
1. We can choose Cloud SQL (Postgres) as our database to gokit-todo application in Google App Engine environment. You must use Cloud SQL proxy postgres driver `cloudsqlpostgres` to connect to Cloud SQL cause Cloud SQL will handle the connection authentication and authorization automatically. ✅
1. We can choose Google Cloud Build to handle CI/CD pipeline workflow and fully integrate with Github repo. There is most important things is you need to grant project default Google Cloud service account `project-number@cloudbuild.gserviceaccount.com` with enough permissions to build and deploy your application. ✅
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/cloudbuild permission.jpg" alt="Google Cloud Build permission">}}

**Architecture**

{{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/placeholder.png" alt="gokit-todo-gae architecture">}}

Above is the whole architecture of our application `gokit-todo-gae` leverage with `gokit-todo` and `gokit-todo-frontend` as git submodule. Using a project with git submodules avoids the overhead of interfering with the application source code.

## Google App Engine

It's very basic setup for our demo scenario. Just enable Google App Engine service in Google Cloud project and we will leverage Google Cloud Build to handle all depoyment jobs

```bash
gcloud app create --region=asia-east1
```

## Cloud SQL

You can create a Cloud SQL instance in Google Cloud Console or `gcloud sql instances create` command. The following gcloud command will create a Cloud SQL.

```bash
gcloud sql instances create todo --database-version=POSTGRES_11 --cpu=1 --memory=3840MiB --region=asia-east1 --root-password=password --storage-size=10GB --storage-type=SSD
```

When Cloud SQL instance is created then we need to crate a demo database.

```bash
gcloud sql databases create todo -i todo
```

It's quick simple step to create a Cloud SQL instance and a demo database via gcloud command. Cloud SQL instance setup with Public IP address as default and you can also setup with Private IP address as your want. Make sure you enable VPC network for your private IP address Cloud SQL instance.

## Google Cloud Build

As previous mentioned that all CI/CD jobs are handled by Google Cloud Build. We need to create a bunch of Cloud Build configurations to build and deploy our application.

1. We added `gokit-todo` and `gokit-todo-frontend` application as project git submodule. We create a Cloud Build configuration for `gokit-todo` and `gokit-todo-frontend` and each of them could trigger another cloud build trigger to deploy application.

      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/submodule trigger.jpg" alt="submodule triggers">}}

      > It's useful tips to trigger Cloud Build trigger by RESTful API in one Cloud Build steps.

      ```bash
      curl -d '{"branchName":"master"}' -X POST -H "Content-type: application/json" \
          -H "Authorization: Bearer $(gcloud config config-helper --format='value(credential.access_token)')" \
          https://cloudbuild.googleapis.com/v1/projects/<gcp-project>/triggers/<cloudbuild-trigger-id>:run
      ```
1. Trigger trigger `gokit-todo` and `gokit-todo-frontend` by `cloudbuild.api.yaml` 及 `cloudbuild.default.yaml`

      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/api default trigger.jpg" alt="api/default triggers">}}

1. It's also to deploy Google App Engine dispatch router `dispatch.yaml` settings through `cloudbuild.dispatch.yaml`

__All of Cloud Build settings__
{{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/cloudbuild-trigger.jpg" alt="Cloud Build triggers">}}

## gokit-todo-gae

**[cage1016/gokit-todo-gae](https://github.com/cage1016/gokit-todo-gae)**

```bash
.
├── api                       // gokit-todo submodule as api service
├── default                   // gokit-todo-frontend as default service
├── .gitmodules
├── cloudbuild.api.yaml       // deploy api service (Manual)
├── cloudbuild.default.yaml   // deploy default service (Manual)
├── cloudbuild.dispatch.yaml  // deploy dispatch.yaml
└── dispatch.yaml             // gokit-todo-gae dispatch yaml
```

You can see above project file structure (gokit-todo-gae) is same as previous architecture diagram. The operation process is as follows

1. Frontend developer push code to Github repo `gokit-todo-frontend` 👉 trigger `gokit-todo-frontend` will be triggered and do their steps job and trigger `gokit-todo-gae-deploy-default` trigger by RESTFul API at last step 👉 trigger `gokit-todo-gae-deploy-default` will build and deploy `gokit-todo-frontend` application to Google App Engine.
   
      __cloudbuild.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-frontend-cloudbuild.yaml.jpg" alt="gokit-todo-frontend cloudbuild.yaml">}}

      __cloudbuild.default.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-gae-cloudbuild.default.yaml.jpg" alt="gokit-todo-gae deploy default service">}}

1. Backend developer push code to Github repo `gokit-todo` 👉 trigger `gokit-todo` will be triggered and do their steps job (application testing) and trigger `gokit-todo-gae-deploy-api` trigger by RESTFul API at last step 👉 trigger `gokit-todo-gae-deploy-api` will build and deploy `gokit-todo` application to Google App Engine.

      __cloudbuild.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-cloudbuild.yaml.jpg" alt="gokit-todo cloudbuild.yaml">}}

      __cloudbuild.api.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-gae-cloudbuild.api.yaml.jpg" alt="gokit-todo-gae deploy api service">}}

1. It's also easy to update Google App Engine dispatch routing setting by push `dispatch.yaml` changed to Github repo `gokit-todo-gae` 👉 trigger `gokit-todo-gae-deploy-dispatch` will update dispatch routing to Google App Engine

      __cloudbuild.dispatch.yaml__
      {{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/cloudbuild.dispatch.yaml.jpg" alt="gokit-todo-gae deploy dispatch.yaml">}}

## summary

Google App Engine is easy to get started with. In particular, for simple applications like this demo, the Google App Engine standard-runtime offers a free tier of 28 instance-hours per day. We can migrate pre Kubernetes application `gokit-todo` and `gokit-todo-frontend` to Google App Engine without any code modified, just leverage with requirement `app.yaml` and Google App Engine could collaborate with GitHub Repo and Google Cloud Build well.

**Q**
Why don't we use Github build-in CI/CD Github action instead of Google Cloud Build ? </br>
**A**
Yes, You could do HTTP request to trigger Google Cloud Build trigger in Github Action but have to handle request authentication by yourself. Google Cloud Build will auto handle those request authentication automatically

{{<image src="/posts/deploy-gokit-todo-to-gae-with-cloud-build-from-github-repo/img/gokit-todo-gae.gif" alt="gokit todo GAE">}}

**source code**
- `gokit-todo` https://github.com/cage1016/gokit-todo
- `gokit-todo-frontend` https://github.com/cage1016/gokit-todo-frontend
- `gokit-todo-gae` https://github.com/cage1016/gokit-todo-gae
