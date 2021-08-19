# K3s Nats tls by openssl


<!--more-->


1. 透過 `k3d` 建立本地 k3s cluster。因為 k3s 也是透過 docker 來摸擬 k8s cluster，所以透過 k3s 本身 port mapping 的配置來對外開放 `8080:80` 及 `4222:31400` 二個 port。 `8080:80` 給 `hashicorp/http-echo` 用。而 `4222:31400` 是對應 NATS nodeport 用

    ```sh
    k3d cluster create dev -p 8080:80@loadbalancer -p 8443:443@loadbalancer -p "4222:31400@server[0]"
    ```

2. 建立 `hashicorp/http-echo`。等待所有的 Pod 及服務就緒後，就可以透過 [http://localhost:8080](http://localhost:8080) 來讀取 `hashicorp/http-echo`

    ```sh
    kubectl run --image hashicorp/http-echo --port 80 echo -- -listen=:80 --text="Hello from echo"
    kubectl expose pod echo --port 80 --target-port 80 --type NodePort --name echo
    cat <<EOF | kubectl apply -f -
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: echo-ingress
      annotations:
        kubernetes.io/ingress.class: traefik
    spec:
      rules:
      - http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              serviceName: echo
              servicePort: 80
    EOF
    ```

3. 下一個步驟就是建立 NATS。因為這一次我們要建立的是有啟用 TLS 的 NATS 服務器。而 TLS 的部份我們採用 self signed 的方式來建立，實際使用就看最終部署的情況來決定是誰來產出 cluster 有需的 TLS，而本 Demo 就方便的方式自建

4. 首先建立所需的 `certificate.conf` 檔案。其中 `IP` 及 `DNS` 可以依照需求進行修改

    ```sh
    cat <<EOF >> certificate.conf
    [req]
    default_bits = 2048
    prompt = no
    default_md = sha256
    req_extensions = req_ext
    distinguished_name = dn
    [dn]
    C = TW
    ST = New Taipei City
    O = Your organization
    CN = localhost
    [req_ext]
    subjectAltName = @alt_names
    [alt_names]
    DNS.1 = kubernetes
    DNS.2 = kubernetes.default
    DNS.3 = kubernetes.default.svc
    DNS.4 = kubernetes.default.svc.cluster
    DNS.5 = kubernetes.default.svc.cluster.local
    DNS.6 = localhost
    IP.1 = 127.0.0.1
    IP.2 = 172.17.34.52
    IP.3 = 192.168.0.15
    [v3_ext]
    authorityKeyIdentifier=keyid,issuer:always
    basicConstraints=CA:FALSE
    keyUsage=keyEncipherment,dataEncipherment
    extendedKeyUsage=serverAuth,clientAuth
    subjectAltName=@alt_names
    EOF
    ```

5. 產生 NATS 服務器有需的 TLS 檔，及之後提供給 Client 使用的 `client1.crt` `client1.key` `ca.crt`  `client2.crt` `client2.key` `ca.crt`。這邊使用的方式是透過 `openssl`，有效日期也是 demo 方便先設為 1000 天。另外我們將啟用 NATS tls verifyAndMap 的功能。所以我們在 `certificate.conf` 的 Subject Alternative Name (SAN)  會指定 email，當 NATS 有開啟 `verifyAndMap` 就會檢查 Subject Alternative Name (SAN)

    ```bash
    cat <<EOF >> gen.sh
    #!/bin/bash

    # Move to root directory...
    mkdir -p keys
    cd keys

    # Generate the Certificate Files
    openssl genrsa -out ca.key 2048
    openssl req -x509 -new -nodes -key ca.key -subj "/CN=localhost" -days 1000 -out ca.crt

    # server
    openssl genrsa -out server.key 2048
    openssl req -new -key server.key -out server.csr -config ../certificate.conf
    openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
        -CAcreateserial -out server.crt -days 1000 \
        -extensions v3_ext -extfile ../certificate.conf

    # client1
    openssl genrsa -out client1.key 2048
    openssl req -new -key client1.key -out client1.csr -config ../certificate.conf
    openssl x509 -req -in client1.csr -CA ca.crt -CAkey ca.key \
        -CAcreateserial -out client1.crt -days 1000 \
        -extensions v3_ext -extfile <(cat $(PWD)/certificate.conf | sed 's/IP.3 = 192.168.0.15/&\nemail.1 = client1@example.com/')

    echo $(PWD)
    # client2
    openssl genrsa -out client2.key 2048
    openssl req -new -key client2.key -out client2.csr -config ../certificate.conf
    openssl x509 -req -in client2.csr -CA ca.crt -CAkey ca.key \
        -CAcreateserial -out client2.crt -days 1000 \
        -extensions v3_ext -extfile ../certificate.conf        
    EOF

    chmod +x ./gen.sh
    ./gen.sh
    ```
1. 我們產生了二組 client tls keypair。我們在產生 client1 的 tls 檔案時加入額外的 Subject Alternative Name (SAN) 資訊 `email:client1@example.com`

    > step: A zero trust swiss army knife for working with X509, OAuth, JWT, OATH OTP, etc. [step-cli | Automate Certificates & Common Cryptography Primitives](https://smallstep.com/cli/)

    ```sh
    step certificate inspect keys/client1.crt 
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 11938605982689757523 (0xa5ae72e347938953)
        Signature Algorithm: SHA1-RSA
            Issuer: CN=localhost
            Validity
                Not Before: Aug 19 03:20:51 2021 UTC
                Not After : May 15 03:20:51 2024 UTC
            Subject: C=TW,ST=New Taipei City,O=Your organization,CN=localhost
            Subject Public Key Info:
                Public Key Algorithm: RSA
                    Public-Key: (2048 bit)
                    Modulus:
                        ...
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Authority Key Identifier:
                    keyid
                X509v3 Basic Constraints:
                    CA:FALSE
                X509v3 Key Usage:
                    Key Encipherment, Data Encipherment
                X509v3 Extended Key Usage:
                    Server Authentication, Client Authentication
                X509v3 Subject Alternative Name:
                    DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster, DNS:kubernetes.default.svc.cluster.local, DNS:localhost
                    email:client1@example.com
                    IP Address:127.0.0.1, IP Address:172.17.34.52, IP Address:192.168.0.15
        Signature Algorithm: SHA1-RSA
            ...    
    ```

    ```sh
    step certificate inspect keys/client2.crt 
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 11938605982689757524 (0xa5ae72e347938954)
        Signature Algorithm: SHA1-RSA
            Issuer: CN=localhost
            Validity
                Not Before: Aug 19 03:20:51 2021 UTC
                Not After : May 15 03:20:51 2024 UTC
            Subject: C=TW,ST=New Taipei City,O=Your organization,CN=localhost
            Subject Public Key Info:
                Public Key Algorithm: RSA
                    Public-Key: (2048 bit)
                    Modulus:
                        ...
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Authority Key Identifier:
                    keyid
                X509v3 Basic Constraints:
                    CA:FALSE
                X509v3 Key Usage:
                    Key Encipherment, Data Encipherment
                X509v3 Extended Key Usage:
                    Server Authentication, Client Authentication
                X509v3 Subject Alternative Name:
                    DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster, DNS:kubernetes.default.svc.cluster.local, DNS:localhost
                    IP Address:127.0.0.1, IP Address:172.17.34.52, IP Address:192.168.0.15
        Signature Algorithm: SHA1-RSA
            ...
    ```



1. 透過 `kubectl` 建立稍後 NATS 服務器所需 TLS 相關的 secret `nats-server-tls`

    ```sh
    kubectl create namespace nats
    kubectl create secret generic nats-server-tls \
      --from-file=ca.crt=./keys/ca.crt \
      --from-file=tls.crt=./keys/server.crt \
      --from-file=tls.key=./keys/server.key \
      --namespace nats
    ```

7. 透過 `helm` 來安裝 NATS 服務器，並指定稍早建立相關的 secret `nats-server-tls`

    > 記得先加入 nats helm chart repo
    >
    > $ `helm repo add nats https://nats-io.github.io/k8s/helm/charts/`
    >
    > $ `helm repo update`
    
    ```sh
    helm install nats nats/nats \
        --set nats.tls.secret.name=nats-server-tls \
        --set nats.tls.ca=ca.crt \
        --set nats.tls.cert=tls.crt \
        --set nats.tls.key=tls.key \
        --set nats.tls.verifyAndMap=true \
        --set auth.enabled=true \
        --set auth.basic.users[0].user=client1@example.com \
        --namespace nats
    ```

8. 基本上作完這一步，如果我們透過 `port-forward` 的方式來 forward `nats/nats-client` `4222` 已經可以讀取到 NATS 服務器。不過使用 `helm` 安裝的 `nats/nats` 預設沒有對外開放服務，又因為使用 k3s cluster 受限於 docker 的原因，我們需要額外指定 NodePort `31400` 對應 k3s `4222:31400` 的設定來給外部讀取

    ```sh
    cat <<EOF >> nats-np.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nats-np
      namespace: nats
      labels:
        app.kubernetes.io/name: nats
        app.kubernetes.io/instance: nats
    spec:
      type: NodePort
      ports:
        - name: tcp-client
          port: 4222
          nodePort: 31400
          targetPort: client
      selector:
        app.kubernetes.io/name: nats
        app.kubernetes.io/instance: nats
    EOF

    kubectl apply -f nats-np.yaml
    ```

9. 讀取 [nats://localhost:4222](nats://localhost:4222)

    ```sh
    nats -s localhost:4222 pub say hi
    ```

    > nats: error: x509: certificate signed by unknown authority, try --help

    由於我們建立 NATS 服務器啟用 TLS，稍早也建立了一組 `client1.crt` `client1.key` `ca.crt`。再連接 NATS 服務器時代入相關的 tls 檔案即可正常操作

    ```sh
    nats -s localhost \
        --tlscert=./keys/client1.crt \
        --tlskey=./keys/client1.key \
        --tlsca=./keys/ca.crt \
        pub say hi
    ```

    因為 NATS 啟用了 verifyAndMap，所以 `client2.crt` `client2.key` `ca.crt` 因為其 Subject Alternative Name (SAN) 沒有包含 email 資訊，所以就算擁有 tls keypair 也沒有權限讀取

    > nats: error: nats: Authorization Violation, try --help

    ```sh
    nats -s localhost \
        --tlscert=./keys/client2.crt \
        --tlskey=./keys/client2.key \
        --tlsca=./keys/ca.crt \
        pub say hi
    ```
