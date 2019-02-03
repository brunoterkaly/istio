

## Configuring the circuit breaker


It is not uncommon during heavy loads for micro service calls to hang without a response. Oftentimes, unnecessarily high levels of resources are consumed while the caller is waiting for the service to respond. Taken too far, the service may be unable to respond at all causing a cascading effect across the application stack composed of a series of micro services.

Services sometimes become unresponsive due to slow network connections, timeout, or the resources being overcommitted or temporarily unavailable. The blocked requests that queue up will therefore unnecessarily hold critical system resources such as memory, threads, and database connections.

The preference in these situations is to faill immediately. This could help prevent a client application from repeatedly trying to execute an operation that's likely to fail.

The idea is that after a series of consecutive failures, a threshold is reached in the circuit breaker trips. This enables the remote service to fail immediately, at which point there will be a timeout that expires the circuit breaker, which is then allowed a limited number of requests to pass through.

With Istio the sidecar proxy (Envoy) takes over and provides circuit breaker services, as a proxy for operations that might fail. The proxy should monitor the number of recent failures that have occurred, and use this information to decide whether to allow the operation to proceed, or simply return an exception immediately.



Create a destination rule to apply circuit breaking settings when calling the httpbin service:

# Circuit Breaker Pattern

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
```

## Apply Circuit Breaker

```
kubectl apply -f circuit-breaker.yml
# If you'd like to clean up
#kubectl delete -f circuit-breaker.yml
```

Verify the destination rule was created correctly. You can output yaml to do so.

```
$ kubectl get destinationrule httpbin -o yaml
```

> Validate the DestinationRule

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"httpbin","namespace":"default"},"spec":{"host":"httpbin","trafficPolicy":{"connectionPool":{"http":{"http1MaxPendingRequests":1,"maxRequestsPerConnection":1},"tcp":{"maxConnections":1}},"outlierDetection":{"baseEjectionTime":"3m","consecutiveErrors":1,"interval":"1s","maxEjectionPercent":100}}}}
  creationTimestamp: 2019-02-02T22:49:18Z
  generation: 1
  name: httpbin
  namespace: default
  resourceVersion: "125034"
  selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/destinationrules/httpbin
  uid: c5fab2ca-273c-11e9-af39-0a58ac1f0a5f
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
      tcp:
        maxConnections: 1
    outlierDetection:
      baseEjectionTime: 3m
      consecutiveErrors: 1
      interval: 1s
      maxEjectionPercent: 100
```
# Using the Fortio Load Testing Service to Trigger Circuit Breaker

Here is a simple GET request that will not trigger the circuit breaker.

```
FORTIO_POD=$(kubectl get pod | grep fortio | awk '{ print $1 }')
echo $FORTIO_POD
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -curl  http://httpbin:8000/get
```
Status 200 comes back.

# Two Steps left - (1) Perform a higher stress load est ing script (2) Query the Istio Proxy for the performance Statistics

This first command 

kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
kubectl exec -it $FORTIO_POD  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending

export FORTIO_POD=$(kubectl get pod | grep fortio | awk '{ print $1 }')
echo $FORTIO_POD
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -curl  http://httpbin:8000/get

root-> echo $FORTIO_POD
fortio-deploy-5d5c6bf6b9-c6jvt

PERFORM SOME LOAD TESTING

root-> kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get

02:23:16 I logger.go:97> Log level is now 3 Warning (was 2 Info)
Fortio 1.0.1 running at 0 queries per second, 2->2 procs, for 20 calls: http://httpbin:8000/get
Starting at max qps with 3 thread(s) [gomax 2] for exactly 20 calls (6 per thread + 2)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
02:23:16 W http_client.go:604> Parsed non ok code 503 (HTTP/1.1 503)
Ended after 41.570256ms : 20 calls. qps=481.11
Aggregated Function Time : count 20 avg 0.005685239 +/- 0.007864 min 0.000337612 max 0.026969444 sum 0.113704781
# range, mid point, percentile, count
>= 0.000337612 <= 0.001 , 0.000668806 , 25.00, 5
> 0.001 <= 0.002 , 0.0015 , 50.00, 5
> 0.002 <= 0.003 , 0.0025 , 55.00, 1
> 0.003 <= 0.004 , 0.0035 , 70.00, 3
> 0.004 <= 0.005 , 0.0045 , 75.00, 1
> 0.005 <= 0.006 , 0.0055 , 80.00, 1
> 0.009 <= 0.01 , 0.0095 , 85.00, 1
> 0.02 <= 0.025 , 0.0225 , 95.00, 2
> 0.025 <= 0.0269694 , 0.0259847 , 100.00, 1
# target 50% 0.002
# target 75% 0.005
# target 90% 0.0225
# target 99% 0.0265756
# target 99.9% 0.0269301
Sockets used: 15 (for perfect keepalive, would be 3)
Code 200 : 8 (40.0 %)
Code 503 : 12 (60.0 %)
Response Header Sizes : count 20 avg 92.1 +/- 112.8 min 0 max 231 sum 1842
Response Body/Total Sizes : count 20 avg 370 +/- 184.1 min 217 max 596 sum 7400
All done 20 calls (plus 0 warmup) 5.685 ms avg, 481.1 qps

root-> kubectl exec -it $FORTIO_POD  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_active: 0
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_failure_eject: 0

// Pending overflow below
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_overflow: 11
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_total: 10


Curl simple and view destination code
Curl at scale and view destination code



kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml)


curl http://httpbin:8000/get -w "%{http_code}\n" 2>&1 | tail -1

kubectl get prods 
Find the `sleep` pod and copy the the whole name


> - kubectl exec -it sleep-7dc47f96b6-7dfld -n bar --container sleep -- /bin/sh





# ========================================
# IGNORE BELOW
# ========================================



# Commands 

kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)
kubectl create -f < (istioctl kube-inject -f bookinfo.yaml)
kubectl create -f <(istioctl kube-inject -f bookinfo.yaml)
kubectl create -f <(istioctl kube-inject -f bookinfo.yaml)
kubectl create -f <(istioctl kube-inject -f $(BOOKINFOYAML))
kubectl create -f <(istioctl kube-inject -f ${BOOKINFOYAML})
kubectl create -f <(istioctl kube-inject -f ${BOOKINFOYAML})
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
history | grep "kubectl.*istioctl"
history | grep "kubectl.*istioctl" > test.txt & vim test.txt
history | grep "kubectl.*istioctl" > test.txt && vim test.txt

kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n bar
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n bar
kubectl apply -f < samples/sleep/sleep.yaml -n legacy
kubectl get pod -l app=sleep -n bar -o jsonpath= {.items..metadata.name}
kubectl get pods sleep-7dc47f96b6-7dfld -n bar -o jsonpath='{.spec.containers[*].name}'
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar --container sleep -- /bin/sh
kubectl create ns foo
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo
kubectl create ns bar
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n bar
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n bar
kubectl create ns legacy
kubectl apply -f samples/httpbin/httpbin.yaml -n legacy
kubectl apply -f samples/sleep/sleep.yaml -n legacy
kubectl get pod -l app=sleep -n bar -o jsonpath={.items..metadata.name}
kubectl get pods sleep-7dc47f96b6-7dfld -n bar -o jsonpath='{.spec.containers[*].name}'
kubectl get services httpbin -o wide -n foo
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar -- /bin/sh
kubectl get policies.authentication.istio.io --all-namespaces
kubectl get meshpolicies.authentication.istio.io
kubectl get destinationrules.networking.istio.io --all-namespaces -o yaml | grep "host:"
kubectl get destinationrules.networking.istio.io --all-namespaces -o yaml
kubectl apply -f default-mesh-policy.yml
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar -- /bin/sh
kubectl apply -f  all-internal-traffic.yml
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar -- /bin/sh
kubectl get pod -l app=sleep -n legacy -o jsonpath={.items..metadata.name}
kubectl exec -it sleep-7dc47f96b6-7dfld -n legacy -- /bin/sh


bash install-helm.sh
cd helm
cd ../helm
cd helmcharts
cd helm-prometheus/
cd /home/azureuser/helmcharts
cd install-helm/
cd learn-helm/
cp show-all.sh /datadrive/helm/project-prometheus
git clone https://github.com/kubernetes/helm.git
grep helm *
grep helm *.sh
grep helm *.sh *.txt
helm
helm delete
helm delete prometheus
helm delete --purge grafana
helm del --purge grafana
helm del --purge prometheus
helm del --purge prometheus && helm del --purge grafana
helm fetch -h
helm fetch incubator/istio
helm fetch stable/prometheus
helm get stable/mysql
helm -h
helm init
helm init --service-account tiller
helm init --service-account tiller --kube-context k8s.example.org
helm init --upgrade
helm inspect prom/node-exporter
helm inspect stable/node-exporter
helm inspect stable/prometheus
helm inspect stable/prometheus | grep -i dae
helm inspect stable/prometheus | vim -
helm install incubator/istio --set rbac.install=false
helm install incubator/istio --set rbac.install=true
helm install install/kubernetes/helm/istio --name istio --namespace istio-system
helm install install/kubernetes/helm/istio --name istio --namespace istio-system   --set global.controlPlaneSecurityEnabled=true   --set grafana.enabled=true   --set tracing.enabled=true   --set kiali.enabled=true
helm install  stable/prometheus
helm list
helm list --all
helm ls
helm ls --all prometheus
helm ls | awk '{print $1}'
helm ls | awk '{print $1}' | vim -
helm repo update
helm search node-exporter
helm search prometheus
helm status
helm template install/kubernetes/helm/istio --name istio --namespace istio-system > $HOME/istio.yaml
helm version
https://storage.googleapis.com/kubernetes-helm/helm-v2.12.1-linux-amd64.tar.gz
if [ -d /home/azureuser/helmcharts ]; then rm -Rf /home/azureuser/helmcharts; fi
init
install install/kubernetes/helm/istio --name istio --namespace istio-system   --set global.controlPlaneSecurityEnabled=true   --set grafana.enabled=true   --set tracing.enabled=true   --set kiali.enabled=true
kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
linux-amd64/helm /usr/local/bin/helm
mkdir helm
mkdir helmcharts
mkdir /helmcharts
mkdir helm-prometheus
mkdir /home/azureuser/helmcharts
mkdir install-helm
mkdir learn-helm
mv charts/ helm
-r gpac helm
rm -Rf helmcharts
rm -Rf /home/azureuser/helmcharts
rm -r helm-prometheus/
snap install helm --classic
vim install-helm.sh
xvzf helm-v2.12.1-linux-amd64.tar.gz





























Run Fortio as a client that will stress the circuit breaker.

```
$ kubectl apply -f <(istioctl kube-inject -f samples/httpbin/sample-client/fortio-deploy.yaml)
```

Log in to the client pod and use the fortio tool to call httpbin. Pass in -curl to indicate that you just want to make one call:

The code below let's us get into the client and run a stressful `curl` command

```
$ FORTIO_POD=$(kubectl get pod | grep fortio | awk '{ print $1 }')
$ kubectl exec -it $FORTIO_POD  -c fortio 
```

You should get `OK`

```
HTTP/1.1 200 OK
....
```

## Clean up rule.

```
$ kubectl delete destinationrule httpbin
```

Shutdown the httpbin service and client:

```
$ kubectl delete deploy httpbin fortio-deploy
$ kubectl delete svc httpbin
```

## Now, itâ€™s time to break something.

### Tripping the circuit breaker

In the DestinationRule settings, you specified maxConnections: 1 and http1MaxPendingRequests: 1. i

Run Fortio as a client that will stress the circuit breaker.

> These rules indicate that if you exceed more than one connection and request concurrently, you should see some failures when the istio-proxy opens the circuit for further requests and connections.

### Call the service with two concurrent connections (-c 2) and send 20 requests (-n 20):

This top one is not going to trip it.

```
$ kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
$ kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
```

```
Fortio 0.6.2 running at 0 queries per second, 2->2 procs, for 5s: http://httpbin:8000/get
Starting at max qps with 3 thread(s) [gomax 2] for exactly 30 calls (10 per thread + 0)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
23:51:51 W http.go:617> Parsed non ok code 503 (HTTP/1.1 503)
Ended after 71.05365ms : 30 calls. qps=422.22
Aggregated Function Time : count 30 avg 0.0053360199 +/- 0.004219 min 0.000487853 max 0.018906468 sum 0.160080597
# range, mid point, percentile, count
>= 0.000487853 <= 0.001 , 0.000743926 , 10.00, 3
> 0.001 <= 0.002 , 0.0015 , 30.00, 6
> 0.002 <= 0.003 , 0.0025 , 33.33, 1
> 0.003 <= 0.004 , 0.0035 , 40.00, 2
> 0.004 <= 0.005 , 0.0045 , 46.67, 2
> 0.005 <= 0.006 , 0.0055 , 60.00, 4
> 0.006 <= 0.007 , 0.0065 , 73.33, 4
> 0.007 <= 0.008 , 0.0075 , 80.00, 2
> 0.008 <= 0.009 , 0.0085 , 86.67, 2
> 0.009 <= 0.01 , 0.0095 , 93.33, 2
> 0.014 <= 0.016 , 0.015 , 96.67, 1
> 0.018 <= 0.0189065 , 0.0184532 , 100.00, 1
# target 50% 0.00525
# target 75% 0.00725
# target 99% 0.0186345
# target 99.9% 0.0188793
Code 200 : 19 (63.3 %)
Code 503 : 11 (36.7 %)
Response Header Sizes : count 30 avg 145.73333 +/- 110.9 min 0 max 231 sum 4372
Response Body/Total Sizes : count 30 avg 507.13333 +/- 220.8 min 217 max 676 sum 15214
All done 30 calls (plus 0 warmup) 5.336 ms avg, 422.2 qps
```
### What to notice

Now you start to see the expected circuit breaking behavior. Only 63.3% of the requests succeeded and the rest were trapped by circuit breaking:

```
Code 200 : 19 (63.3 %)
Code 503 : 11 (36.7 %)
```

### Query the istio-proxy stats to see more:

```
$ kubectl exec -it $FORTIO_POD  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
```

Notice the output

```
cluster.outbound|80||httpbin.springistio.svc.cluster.local.upstream_rq_pending_active: 0
cluster.outbound|80||httpbin.springistio.svc.cluster.local.upstream_rq_pending_failure_eject: 0
cluster.outbound|80||httpbin.springistio.svc.cluster.local.upstream_rq_pending_overflow: 12
cluster.outbound|80||httpbin.springistio.svc.cluster.local.upstream_rq_pending_total: 39
```

## Results

You can see 12 for the upstream_rq_pending_overflow value which means 12 calls so far have been flagged for circuit breaking.
