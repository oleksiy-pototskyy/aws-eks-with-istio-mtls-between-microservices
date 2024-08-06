# mTLS with Istio on AWS EKS

This repository demonstrates how to enable mutual TLS (mTLS) between two services running on an Amazon EKS cluster using Istio.

## Prerequisites

- AWS EKS cluster
- `kubectl` and `helm` installed and configured.

## Steps to Deploy

### 1. Install Istio

Download and install Istio:

```sh
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.22.3
export PATH=$PWD/bin:$PATH
istioctl x precheck
istioctl install --set profile=demo -y
```

### 2. Create namespace and deploy demo application 

```sh
kubectl create ns demo
kubectl label namespace demo istio-injection=enabled
helm install demo ./demo
```

### 3. Enable mTLS 

mTLS is already enabled by default in the Helm chart with the peer-authentication.yaml and destination-rule.yaml templates.

### 4. Test mTLS 

Ensure that the PeerAuthentication resource is configured correctly: 

```shell
> kubectl -n demo get peerauthentication -o yaml

apiVersion: v1
items:
- apiVersion: security.istio.io/v1
  kind: PeerAuthentication
  metadata:
    annotations:
      meta.helm.sh/release-name: demo
      meta.helm.sh/release-namespace: demo
    creationTimestamp: "2024-08-06T19:49:29Z"
    generation: 1
    labels:
      app.kubernetes.io/managed-by: Helm
    name: demo
    namespace: demo
    resourceVersion: "41063"
    uid: 6d99bafa-0ac9-493b-8b0f-8e195584e28c
  **spec:
    mtls:
      mode: STRICT**
kind: List
```

Ensure that the DestinationRule resource is configured for mTLS

```shell
> kubectl -n demo get destinationrule -o yaml

apiVersion: v1
items:
- apiVersion: networking.istio.io/v1
  kind: DestinationRule
  metadata:
    annotations:
      meta.helm.sh/release-name: demo
      meta.helm.sh/release-namespace: demo
    creationTimestamp: "2024-08-06T19:49:19Z"
    generation: 1
    labels:
      app.kubernetes.io/managed-by: Helm
    name: app1-mtls
    namespace: demo
    resourceVersion: "41024"
    uid: 95b2c765-e364-4265-b666-503e78f15650
  spec:
    host: app1.default.svc.cluster.local
    **trafficPolicy:
      tls:
        mode: ISTIO_MUTUAL**
- apiVersion: networking.istio.io/v1
  kind: DestinationRule
  metadata:
    annotations:
      meta.helm.sh/release-name: demo
      meta.helm.sh/release-namespace: demo
    creationTimestamp: "2024-08-06T19:49:19Z"
    generation: 1
    labels:
      app.kubernetes.io/managed-by: Helm
    name: app2-mtls
    namespace: demo
    resourceVersion: "41025"
    uid: ee81e912-07f8-452a-b274-a2b543e2caea
  spec:
    host: app2.default.svc.cluster.local
    **trafficPolicy:
      tls:
        mode: ISTIO_MUTUAL**
kind: List
```

We can also check Istio proxy logs via `kubectl` and use `istioctl` to inspect the configuration. 

