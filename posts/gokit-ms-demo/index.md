# Gokit microservice demo


<!--more-->

小弟在 GDG Devfest 2019 分享過 [Build go kit microservices at kubernetes with ease](https://www.slideshare.net/cagechung/gdg-devfest-2019-build-go-kit-microservices-at-kubernetes-with-ease)，Gokit 是一個建立 microservice 的 toolkit，Gokit 提出 `Transport` `Endpoint` `Service` 三種概念來幫助開始者進行架構分離，對單一微服務進行架構強制分離或許會增加程式碼的閱讀性，不過對一定規模的微服務來說，一致性的程式架構分離反而會增加多人開發的效率

## cage1016/ms-demo

| Service | Description           |
| ------- | --------------------- |
| add     | Expose Sum method     |
| tictac  | Expose Tic/Tac method |

[cage1016/ms-demo: gokit microservice demo](https://github.com/cage1016/ms-demo) 提供了使用 gokit 建立的 kubernetes/istio 的 manifest, 可以讓使用者快速的練習基於 kubernetes/istio 來搭建 gokit 微服務

### Install

1. Run `skaffold run` (first time will be slow)
2. Set the `ADD_HTTP_LB_URL/ADD_GRPC_LB_URL` & `TICTAC_HTTP_LB_URL/TICTAC_GRPC_LB_URL` environment variable in your shell to the public IP/port of the Kubernetes loadBalancer
    ```shell
    export ADD_HTTP_LB_PORT=$(kubectl get service add-external -o jsonpath='{.spec.ports[?(@.name=="http")].port}')
    export ADD_GRPC_LB_PORT=$(kubectl get service add-external -o jsonpath='{.spec.ports[?(@.name=="grpc")].port}')
    export ADD_LB_HOST=$(kubectl get service add-external -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
    export ADD_HTTP_LB_URL=$ADD_LB_HOST:$ADD_HTTP_LB_PORT
    export ADD_GRPC_LB_URL=$ADD_LB_HOST:$ADD_GRPC_LB_PORT
    echo $ADD_HTTP_LB_URL
    echo $ADD_GRPC_LB_URL

    export TICTAC_HTTP_LB_PORT=$(kubectl get service tictac-external -o jsonpath='{.spec.ports[?(@.name=="http")].port}')
    export TICTAC_GRPC_LB_PORT=$(kubectl get service tictac-external -o jsonpath='{.spec.ports[?(@.name=="grpc")].port}')
    export TICTAC_LB_HOST=$(kubectl get service tictac-external -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
    export TICTAC_HTTP_LB_URL=$TICTAC_LB_HOST:$TICTAC_HTTP_LB_PORT
    export TICTAC_GRPC_LB_URL=$TICTAC_LB_HOST:$TICTAC_GRPC_LB_PORT
    echo $TICTAC_HTTP_LB_URL
    echo $TICTAC_GRPC_LB_URL
    ```
3. Access by command
    - sum method
    ```shell
    curl -X POST $ADD_HTTP_LB_URL/sum -d '{"a": 1, "b":1}'
    
    or
    
    grpcurl -d '{"a": 1, "b":1}' -plaintext -proto ./pb/add/add.proto $ADD_GRPC_LB_URL pb.Add.Sum
    ```
    - tic method
    ```shell
    curl -X POST $TICTAC_HTTP_LB_URL/tic
    
    or
    
    grpcurl -plaintext -proto ./pb/tictac/tictac.proto $TICTAC_GRPC_LB_URL pb.Tictac.Tic
    ```
    - tac method
    ```shell
    curl $TICTAC_HTTP_LB_URL/tac
    
    or
    
    grpcurl -plaintext -proto ./pb/tictac/tictac.proto $TICTAC_GRPC_LB_URL pb.Tictac.Tac
    ```
4. Apply istio manifests `kubectl apply -f deployments/istio-manifests`
5. Set the `GATEWAY_HTTP_URL/GATEWAY_GRPC_URL` environment variable in your shell to the public IP/port of the Istio Ingress gateway.
    ```shell
    export INGRESS_HTTP_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    export INGRESS_GRPC_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
    export GATEWAY_HTTP_URL=$INGRESS_HOST:$INGRESS_HTTP_PORT
    export GATEWAY_GRPC_URL=$INGRESS_HOST:$INGRESS_GRPC_PORT
    echo $GATEWAY_HTTP_URL
    echo $GATEWAY_GRPC_URL
    ```
7. Access by command
    - sum method
    ```shell
    curl -X POST $GATEWAY_HTTP_URL/api/v1/add/sum -d '{"a": 1, "b":1}'
    
    or
    
    grpcurl -d '{"a": 1, "b":1}' -plaintext -proto ./pb/add/add.proto $GATEWAY_GRPC_URL pb.Add.Sum
    ```
    - tic method
    ```shell
    curl -X POST $GATEWAY_HTTP_URL/api/v1/tictac/tic
    
    or
    
    grpcurl -plaintext -proto ./pb/tictac/tictac.proto $GATEWAY_GRPC_URL pb.Tictac.Tic
    ```
    - tac method
    ```shell
    curl $GATEWAY_HTTP_URL/api/v1/tictac/tac
    
    or
    
    grpcurl -plaintext -proto ./pb/tictac/tictac.proto $GATEWAY_GRPC_URL pb.Tictac.Tac
    ```


### CleanUP

`skaffold delete`

or 

`kubectl delete -f deployments/istio-manifests`
