# NATS quick expose for local test


<!--more-->

## bitnami/nats

> https://github.com/bitnami/charts/tree/master/bitnami/nats/#installing-the-chart

為了本機端使用 KIND/K3d 為基礎開發 Kubernetes 有 NATS 介接需求時，可以透過 `Helm` 來快速部署且只有開發需求，就可以使用 `bitnami/nats` 的版本來使用。所以在 NATS 的 Expose 方式使用 `NodePort` 將 NATS client `4222` 指定給 `31400`

```
helm install nats \
  --set auth.enabled=false \
  --set client.service.type=NodePort \
  --set client.service.nodePort=31400 \
  bitnami/nats --create-namespace --namespace nats
```

待 `Helm` 安裝完 bitnami/nats 可以檢視 nats-client

```bash
$ k -n nats get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
nats-client       NodePort    10.96.104.5     <none>        4222:31400/TCP      20m
nats-cluster      ClusterIP   10.96.97.123    <none>        6222/TCP            20m
nats-headless     ClusterIP   None            <none>        4222/TCP,6222/TCP   20m
nats-monitoring   ClusterIP   10.96.124.235   <none>        8222/TCP            20m
```

## Kubernetes Cluster

### KIND

```bash
KIND_NATS_PORT=4222
cat <<EOF >> kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind-testbed
nodes:
- role: control-plane
  # port forward 80 on the host to 80 on this node
  extraPortMappings:
  - containerPort: 31400
    hostPort: ${KIND_NATS_PORT}
    # optional: set the bind address on the host
    # 0.0.0.0 is the current default
    # optional: set the protocol to one of TCP, UDP, SCTP.
    # TCP is the default
    protocol: TCP
EOF

# create local kind cluster
kind create cluster --config kind-config.yaml
```

### K3d

```bash
K3D_NATS_PORT=4223
cat <<EOF >> k3d-config.yaml
apiVersion: k3d.io/v1alpha2
name: k3d-testbed
kind: Simple
servers: 1
ports:
  - port: ${K3D_NATS_PORT}:31400
    nodeFilters:
      - server[*]
EOF

# create local k3d cluster
k3d cluster create --config k3d-config.yaml
```

### Docker

這時候我們就可以輕鬆透過 `:::4223` 及 `0.0.0.0:4222` 來連接二個 k8s 內部的 NATS 服務

```bash
$ docker ps
... PORTS                                                NAMES
... 0.0.0.0:4223->31400/tcp, :::4223->31400/tcp          k3d-k3d-testbed-server-0
... 127.0.0.1:65264->6443/tcp, 0.0.0.0:4222->31400/tcp   kind-testbed-control-plane
```
