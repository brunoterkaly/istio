

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

Fortio will be learned for the load testing. You can learn more at Github.

Fortio runs at a specified query per second (qps) and records an histogram of execution time and calculates percentiles (e.g. p99 ie the response time such as 99% of the requests take less than that number (in seconds, SI unit)). It can run for a set duration, for a fixed number of calls, or until interrupted (at a constant target QPS, or max speed/load per connection/thread).

The name fortio comes from greek φορτίο which means load/burden.

![fortio](./images/fortio.png)

Here is a simple GET request that will not trigger the circuit breaker.

```
FORTIO_POD=$(kubectl get pod | grep fortio | awk '{ print $1 }')
echo $FORTIO_POD
fortio-deploy-5d5c6bf6b9-c6jvt
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -curl  http://httpbin:8000/get
Status 200 comes back.
```

# Two Steps left - (1) Perform a higher stress load est ing script (2) Query the Istio Proxy for the performance Statistics

This first command like

```
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
```

```
kubectl exec -it $FORTIO_POD  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
```

PERFORM SOME LOAD TESTING

```
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
```

You can see the detailed output.

```
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
```

This next command checks the istio-proxy for some stats.

```
kubectl exec -it $FORTIO_POD  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
```
The results:

```
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_active: 0
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_failure_eject: 0

// Pending overflow below
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_overflow: 11
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_total: 10
```
