
# Create an HTTPS service with Istio sidecar with mutual TLS enabled

**Control Plane with TLS enabled** - You need to deploy Istio control plane with mutual TLS enabled. If you have istio control plane with mutual TLS disabled installed, please delete it. For example, if you followed the quick start:

```
$ kubectl delete -f install/kubernetes/istio-demo.yaml
```

And wait for everything is down, i.e., there is no pod in control plane namespace (istio-system).

```
$ kubectl get pod -n istio-system
No resources found.
```

Then deploy the Istio control plane with mutual TLS enabled:

```
$ kubectl apply -f install/kubernetes/istio-demo-auth.yaml
```

Make sure everything is up and running:

```
$ kubectl get po -n istio-system
NAME                                       READY     STATUS      RESTARTS   AGE
grafana-6f6dff9986-r6xnq                   1/1       Running     0          23h
istio-citadel-599f7cbd46-85mtq             1/1       Running     0          1h
istio-cleanup-old-ca-mcq94                 0/1       Completed   0          23h
istio-egressgateway-78dd788b6d-jfcq5       1/1       Running     0          23h
istio-ingressgateway-7dd84b68d6-dxf28      1/1       Running     0          23h
istio-mixer-post-install-g8n9d             0/1       Completed   0          23h
istio-pilot-d5bbc5c59-6lws4                2/2       Running     0          23h
istio-policy-64595c6fff-svs6v              2/2       Running     0          23h
istio-sidecar-injector-645c89bc64-h2dnx    1/1       Running     0          23h
istio-statsd-prom-bridge-949999c4c-mv8qt   1/1       Running     0          23h
istio-telemetry-cfb674b6c-rgdhb            2/2       Running     0          23h
istio-tracing-754cdfd695-wqwr4             1/1       Running     0          23h
prometheus-86cb6dd77c-ntw88                1/1       Running     0          23h
servicegraph-5849b7d696-jrk8h              1/1       Running     0          23h
```

Then redeploy the HTTPS service and sleep service

```
$ kubectl delete -f <(bin/istioctl kube-inject -f samples/sleep/sleep.yaml)
$ kubectl apply -f <(bin/istioctl kube-inject -f samples/sleep/sleep.yaml)
$ kubectl delete -f <(bin/istioctl kube-inject -f samples/https/nginx-app.yaml)
$ kubectl apply -f <(bin/istioctl kube-inject -f samples/https/nginx-app.yaml)
```

Make sure the pod is up and running

```
$ kubectl get pod
NAME                              READY     STATUS    RESTARTS   AGE
my-nginx-9dvet                    2/2       Running   0          1h
sleep-77f457bfdd-hdknx            2/2       Running   0          18h
```

And run

```
$ kubectl exec $(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name}) -c sleep -- curl https://my-nginx -k
```

Output:

```
...
<h1>Welcome to nginx!</h1>
...
```

The reason is that for the workflow “sleep -> sleep-proxy -> nginx-proxy -> nginx”, the whole flow is L7 traffic, and there is a L4 mutual TLS encryption between sleep-proxy and nginx-proxy. In this case, everything works fine.

However, if you run this command from istio-proxy container, it will not work. And it should not.

$ kubectl exec $(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name}) -c istio-proxy -- curl https://my-nginx -k
curl: (35) gnutls_handshake() failed: Handshake failed
command terminated with exit code 35

The reason is that for the workflow “sleep-proxy -> nginx-proxy -> nginx”, nginx-proxy is expected mutual TLS traffic from sleep-proxy. In the command above, sleep-proxy does not provide client cert. As a result, it won’t work. Moreover, even sleep-proxy provides client cert in above command, it won’t work either since the traffic will be downgraded to http from nginx-proxy to nginx.

### Cleanup
```
$ kubectl delete -f samples/sleep/sleep.yaml
$ kubectl delete -f samples/https/nginx-app.yaml
$ kubectl delete configmap nginxconfigmap
$ kubectl delete secret nginxsecret
```
