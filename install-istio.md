# Istio Installation

## The steps to install Istio are:

1. Have a Kubernetes cluster (AKS) ready with kubectl configured.
2. Download Istio
2. Install Helm and Tiller
3. Download the Release

### Download Istio

#### https://istio.io/docs/setup/kubernetes/download-release/

```bash
curl -L https://git.io/getLatestIstio | sh -
cd istio-1.0.5
export PATH=$PWD/bin:$PATH
```
I updated my `.bashrc` file as follows:

```bash
export ISTIO="/root/istio/istio-1.0.5/bin"
export PATH="$PATH:$GOPATH/bin:$GOROOT/bin:$ISTIO"
source <(kubectl completion bash)
```
You may have a different path as I am running as `root` locally.

### Installation with Helm

If a `service account` has not already been installed for Tiller, install one:

```bash
kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
```

#### Install Tiller on your cluster with the service account:

```bash
 helm init --service-account tiller
 ```

#### Install Istio

```bash
helm install install/kubernetes/helm/istio --name istio --namespace istio-system
 ```

In my `.bashrc` file I added the ISTIO environment variable.

```bash
export ISTIO="/root/istio/istio-1.0.5/bin"
export PATH="$PATH:$ISTIO"
source <(kubectl completion bash)
```

## Install with Helm and Tiller via helm install

You can install Tiller on your cluster with the service account:

```bash
helm init --service-account tiller
```

### Install Istio:

```bash
helm install install/kubernetes/helm/istio --name istio-minimal --namespace istio-system \
  --set security.enabled=false \
  --set ingress.enabled=false \
  --set gateways.istio-ingressgateway.enabled=false \
  --set gateways.istio-egressgateway.enabled=false \
  --set galley.enabled=false \
  --set sidecarInjectorWebhook.enabled=false \
  --set mixer.enabled=false \
  --set prometheus.enabled=false \
  --set global.proxy.envoyStatsd.enabled=false \
  --set pilot.sidecar=false
```

**Ensure the istio-pilot-Kubernetes pod is deployed** - Ensure its container is up and running:

```bash
$ kubectl get pods -n istio-system
```
You should see these results:


```bash
NAME                                     READY     STATUS    RESTARTS   AGE
istio-pilot-58c65f74bc-2f5xn             1/1       Running   0          1m
```

```bash
 ```
```bash
 ```

```bash
 ```
```bash
 ```