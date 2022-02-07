# Telepresence 2 Have a Tried


<!--more-->

在開發 Kuberentes 應用程式時使用 [Skaffold](https://github.com/GoogleContainerTools/skaffold) 應該是基本操作了，Skaffold 可以幫忙加速開發的速度 (修改程式碼 → 構建 container image → push container image to registry (optional) → 部署至 Kubernets Cluster)，至於是否搭配 `Helm` 還是直接操作 `yaml` 就看個人喜好來決定

在 Debug 部份，Skaffold 也支援 Remote container debug 的功能 (之前的文章請參照 [Skaffold debug goland](https://kaichu.io/posts/skaffold-debug-goland/))。雖然 Skaffold debug 很方便，不過當 Kubernetes 的應用一多時就沒有辦法在本機端完整的重現有所的服務，這時候開源的 Telepresence 就是一個很好的幫手

### Telepresence

{{<image src="/posts/telepresence-2-have-a-tried/img/bird-on-bricks.svg">}}

> Local development against a remote Kubernetes or OpenShift cluster

> ** Note: Telepresence 1 is being replaced by our even better [Telepresence 2](https://github.com/telepresenceio/telepresence/tree/release/v2). Please try Telepresence 2 first and report any issues as we expect this will be the default by Q2 2021. **

{{<image src="/posts/telepresence-2-have-a-tried/img/telepresence-architecture.inline.svg" caption="telepresence architecture">}} (ref: https://www.getambassador.io/docs/telepresence/latest/reference/architecture/)

(這邊以 Telepresence 2 為主) 基本上來說 Telepresence 透過 Traffic agent 將所有目標服務的流量/特定 header("x-telepresence-intercept-id") 請求重導至本機中，這樣我們就針對單一服務進行快速的進行開發，同時地機端如果也連接至 Kubernets Cluster 其他的資源也是相通的

```bash
telepresence version
Client v2.2.1 (api v3)
Daemon v2.2.0 (api v3)
```


```bash
Usage:
  telepresence [command]

Available Commands:
  Session Commands:
    connect              Connect to a cluster
    login                Authenticate to Ambassador Cloud
    logout               Logout from Ambassador Cloud
    license              Get License from Ambassador Cloud
    status               Show connectivity status
    quit                 Tell telepresence daemon to quit
  Traffic Commands:
    list                 List current intercepts
    intercept            Intercept a service
    leave                Remove existing intercept
    preview              Create or remove preview domains for existing intercepts
  Other Commands:
    version              Show version
    uninstall            Uninstall telepresence agents and manager
    dashboard            Open the dashboard in a web page
    current-cluster-id   Get cluster ID for your kubernetes cluster

```

Telepresence 1.0 跟 2.0 差異很大，2.0 是以 Golang 重新改寫，官方也建議從 2.0 開始，不過 2.0 目前還沒有完全開發完成

```bash
NAMESPACE       NAME                                        READY   STATUS    RESTARTS   AGE
ambassador      traffic-manager-78f4f95c7d-z62lm            1/1     Running   1          33h
```

第一次操作 Telepresence，會在 Kubernets 中建立 traffic-manager, Traffic-manager 是 Telepresence 2.0 的核心組件也是負責本地 Telepresence Demaons 及目標 Pod 中 Traffice Agent 的溝通

```bash
kubectl delete svc,deploy -n ambassador traffic-manager
``` 

必要的時候可以進行刪除，下一次 Telepresence 重新連線的時候會重建


### cage1016/ms-demo

> gokit microservice demo

接下來 Demo 就以 [cage1016/ms-demo](https://github.com/cage1016/ms-demo) 中的 gokit microservice demo 來進行操作

| Service | Method |Description                                        |
| ------- | ------ |-------------------------------------------------- |
| add     | Sum    |Expose Sum(a,b) method                             |
| tictac  | Tic    |Expose Tic method (incrase value by Add Sum GRPC ) |
| tictac  | Tac    |Expose Tac method (recive value)                   |

{{<image src="/posts/telepresence-2-have-a-tried/img/demo-architecture.jpg" caption="gokit microservice demo architecture">}}

#### all TCP connections

1. 準個一個可用的 K8s cluster，我們這邊使用 Mac docker-desktop 的 k8s cluster
1. 部署 [cage1016/ms-demo: gokit microservice demo](https://github.com/cage1016/ms-demo) `Add`, `Tictac` 微服務

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/cage1016/ms-demo/master/deployments/kubernetes-manifests-all.yaml
    ```

    ```bash
    kubectl get po
    NAME                      READY   STATUS    RESTARTS   AGE
    add-68b7f4b486-cqf4m      1/1     Running   0          17m
    tictac-85f698c88f-cw6mg   2/2     Running   1          17m
    ```

1. Expose service，這邊使用的方式為 LoadBalancer

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/cage1016/ms-demo/master/deployments/lb-all.yaml
    ```

    ```bash
    kubectl get svc
    NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
    add               ClusterIP      10.97.120.99    <none>        80/TCP,8000/TCP                 18m
    add-external      LoadBalancer   10.101.69.142   localhost     8180:31605/TCP,8181:32609/TCP   5m57s
    kubernetes        ClusterIP      10.96.0.1       <none>        443/TCP                         5d5h
    tictac            ClusterIP      10.96.191.66    <none>        80/TCP,8000/TCP                 18m
    tictac-external   LoadBalancer   10.100.63.131   localhost     9190:30349/TCP,9191:30063/TCP   5m57s
    ```

1. 設定攔截器，intercept `Add` service name `grpc` 至本機中 `10121` (GRPC) service，等待指令完成後中顯示會攔截 add service 所有流量至 127.0.0.1:10121

    ```bash
    telepresence intercept add --service add --port 10121:grpc
    Using Deployment add
    intercepted
        Intercept name         : add
        State                  : ACTIVE
        Workload kind          : Deployment
        Destination            : 127.0.0.1:10121
        Service Port Identifier: grpc
        Volume Mount Error     : macFUSE 4.0.5 or higher is required on your local machine
        Intercepting           : all TCP connections
    ```

1. 這時候我們可以看到 add Pod 中已經被置入 `traffic-agent`

    ```bash
    kubectl iexec add
    Use the arrow keys to navigate: ↓ ↑ → ←
    ? Select Container:
      Container: ▸ add
      Container: traffic-agent
    ```

1. 在 [cage1016/ms-demo](https://github.com/cage1016/ms-demo) 中進行 Debug，VSCode 中 `lanuch.json` 加入 Debug 設定後執行

    ```json
    {
        "configurations": [
            {
                "name": "add",
                "type": "go",
                "request": "launch",
                "mode": "auto",
                "program": "${workspaceFolder}/cmd/add",
                "env": {
                    "QS_LOG_LEVEL": "info",
                    "QS_HTTP_PORT": "10120",
                    "QS_GRPC_PORT": "10121",
                }
            },
        ]
    }
    ```

    {{<image src="/posts/telepresence-2-have-a-tried/img/debug.png" caption="debug">}}

1. 設定 `Add` 服務中斷點 

    {{<image src="/posts/telepresence-2-have-a-tried/img/debug-2.png" caption="debug">}}

1. 獲取 `Tictac` 服務的 URL

    ```bash
    TICTAC_HTTP_EXTERNAL_PORT=$(kubectl get service tictac-external -o jsonpath='{.spec.ports[?(@.name=="http")].port}')
    TICTAC_EXTERNAL_HOST=$(kubectl get service tictac-external -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
    # TICTAC_EXTERNAL_HOST=$(kubectl get service tictac-external -o jsonpath='{.status.loadBalancer.ingress[0].ip}') # unmark if necessary
    TICTAC_HTTP_EXTERNAL_URL=$TICTAC_EXTERNAL_HOST:$TICTAC_HTTP_EXTERNAL_PORT
    echo $TICTAC_HTTP_EXTERNAL_URL
    ```

1. 使用 curl 進行請求

    ```bash
    curl -X POST $TICTAC_HTTP_EXTERNAL_URL/tic
    ```

1. VSCode 中執行中的 Add service 會成功攔截至流量並停在我們設置的中斷點中

    {{<image src="/posts/telepresence-2-have-a-tried/img/debug-3.png" caption="debug">}}


####  header("x-telepresence-intercept-id")

Telepresence 除了全流量的攔截之外也支援特定的請求，基本上的操作步驟是一樣的，差異的部份需要先進行 Telepresence login

1. 進行 Telepresence login

    ```bash
    Telepresence login
    ```

1.  (Optional)，如果已經跑過一次 intercept，記得先退出當前的 intercept

    ```bash
    Telepresence leave add
    ```


1.  一樣進行設定攔截器的配置並加上 `--preview-url=false`，我們並不需在 Ambassador 上產生 preview url

    > 需要 Telepresence login 操作 intercept 才會有特定請求，不然怎麼操作都會是 all TCP connections

    ```
    telepresence intercept add --service add --port 10121:grpc --preview-url=false
    Using Deployment add
    intercepted
        Intercept name         : add
        State                  : ACTIVE
        Workload kind          : Deployment
        Destination            : 127.0.0.1:10121
        Service Port Identifier: grpc
        Volume Mount Error     : macFUSE 4.0.5 or higher is required on your local machine
        Intercepting           : HTTP requests that match all of:
          header("x-telepresence-intercept-id") ~= regexp("5b771ea3-8f6a-4be1-82f3-675aa1e28840:add")
    ```

1.  這時候再使用 curl 進行請求並加上特定的 (x-telepresence-intercept-id) 才會進行攔截，沒有加上 Header 的請求反之不理

    ```bash
    curl -H 'x-telepresence-intercept-id: 5b771ea3-8f6a-4be1-82f3-675aa1e28840:add' -X POST $TICTAC_HTTP_EXTERNAL_URL/tic
    ```

1. VSCode 攔截特定請求

    > 這邊一個成功的因素是 tictac tic 呼叫 add sum GRPC 時一同把 header 的 x-telepresence-intercept-id 加到 context 才可以

    {{<image src="/posts/telepresence-2-have-a-tried/img/debug-4.png" caption="debug">}}


### 結論

Teleresence 2 可以有效的降低多個微服務的 debug 體驗，只攔截特定請求更是方便，不需要擔心把微服務咬死，不過目前開發還在進行中，Github community 回報的 issue 還是不少，還是值得一玩。大家可以至 [cage1016/ms-demo: gokit microservice demo](https://github.com/cage1016/ms-demo) 中 clone 有所的程式照著操作看看體驗看看
