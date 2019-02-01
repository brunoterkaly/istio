# Walkthrough to Understand Mutual TLS

![Mutual TLS](./images/mutual-tls.png)

> You can follow along here as well: https://istio.io/docs/tasks/security/authn-policy/

The purpose of this section is to implement Transport Authentication. The goal is to limit traffic in and out of Kubernetes namespaces.

x509 Certificates will be used for this purpose, supporting two-way authentication with a key management system to automate key and certificate generation, distribution, and rotation.

This will provide privacy & integrity of data between two applications communicating with each other. This will also provide secured connections for both communications done over Internet and in private cluster.

Two namespaces will be provided so we can show that even with Istio installed, Mutual TLS support isn't the default. We  have to turn it on. So in the first part will will show that services across namespaces can talk to each other. 

We will then establish a `Destination Rule` for the ns=bar that disables traffic to the other namespaces, including the ns=legacy.

If attempts are made by apps in the ns=bar to reach other apps in other namespaces, they will be prohibited, getting an HTTP code of 56 (command terminated with exit code 56), instead of 200 (OK).

## Key commands


Applications
    
- httpbin
- sleep

Namespaces

 - foo
    - httpbin
    - sleep
 - bar
    - httpbin
    - sleep
 - legacy
    - httpbin
    - sleep

 **Provisioning apps (httpbin, sleep) for Namespace = foo**

Create a  namespace

- kubectl create ns foo
- ns=foo

Provision httpbin in Namespace=foo

- kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo
- Service=httpbin, Pod=httpbin, Namespace=foo
- Includes Istio sidecar container

Provision sleep in Namespace=foo

- kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo
- Service=sleep, Pod=sleep, Namespace=foo
- Includes Istio sidecar container

**Provisioning apps (httpbin, sleep) for Namespace = bar**

Create a  namespace

- kubectl create ns bar
- ns=bar

Provision httpbin in Namespace=bar

- kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n bar
- Service=httpbin, Pod=httpbin, Namespace=bar
- Includes Istio sidecar container

Provision sleep in Namespace=bar

- kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n bar
- Service=sleep, Pod=sleep, Namespace=bar
- Includes Istio sidecar container


**Provisioning apps (httpbin, sleep) for Namespace = legacy**

Create a  namespace

- kubectl create ns legacy
- ns=legacy

Provision httpbin in Namespace=legacy

- kubectl apply -f samples/httpbin/httpbin.yaml -n legacy
- Service=httpbin, Pod=httpbin, Namespace=legacy
- Includes Istio sidecar container

Provision sleep in Namespace=legacy

- kubectl apply -f < samples/sleep/sleep.yaml -n legacy
- Service=sleep, Pod=sleep, Namespace=legacy
- Does NOT include Istio sidecar container



**Some key commands**

> Get the name of a pod
> - kubectl get pod -l app=sleep -n bar -o jsonpath= {.items..metadata.name}
> 
>  Get the containers in a pod
> - kubectl get pods sleep-7dc47f96b6-7dfld -n bar -o jsonpath='{.spec.containers[*].name}'
> 
>  Get information for Kubernetes Service INTERNAL endpoint
>   kubectl get services httpbin -o wide -n foo
> - Internal Endpoint = Service Name + Namespace + Port
> 
>  Remote into a container that is in a specific pod and namespace
> - kubectl exec -it sleep-7dc47f96b6-7dfld -n bar --container sleep -- /bin/sh
> 
> Issue Curl command against Internal Endpoint of httpbin service
> - curl http://httpbin.foo:8000 -w "%{http_code}\n"

**Provision 3 namespaces: (1) foo; (2) bar; (3) legacy. foo and bar have Istio support. `Legacy` does not** 

Let's provision all the apps in all the namespaces.

```bash
$ kubectl create ns foo
$ kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo
$ kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo
$ kubectl create ns bar
$ kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n bar
$ kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n bar
$ kubectl create ns legacy
$ kubectl apply -f samples/httpbin/httpbin.yaml -n legacy
$ kubectl apply -f samples/sleep/sleep.yaml -n legacy
```

The output from the above commands.

```bash
namespace/foo created
   service/httpbin created
   deployment.extensions/httpbin created

   service/sleep created
   deployment.extensions/sleep created

namespace/bar created
   service/httpbin created
   deployment.extensions/httpbin created

   service/sleep created
   deployment.extensions/sleep created

namespace/legacy created
   service/httpbin created
   deployment.extensions/httpbin created

   service/sleep created
   deployment.extensions/sleep created

```

## Let's review the pods and services deployed

Let's make sure we understand the httpbin app and the sleep app. Here are the yaml files. Be aware that the Envoy proxy (sidecar) will be added to the namespaces foo and bar. 

Below you can see these are simple apps. httpbin is a Python web site and sleep gives us the ability to issue `curl` commands.

### httpbin.yaml

```yml
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
spec:
  ports:
  - name: http
    port: 8000
  selector:
    app: httpbin
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: httpbin
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      containers:
      - image: docker.io/citizenstig/httpbin
        imagePullPolicy: IfNotPresent
        name: httpbin
        ports:
        - containerPort: 8000
```


### sleep.yaml

```yml
apiVersion: v1
kind: Service
metadata:
  name: sleep
  labels:
    app: sleep
spec:
  ports:
  - port: 80
    name: http
  selector:
    app: sleep
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: sleep
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sleep
    spec:
      containers:
      - name: sleep
        image: pstauffer/curl
        command: ["/bin/sleep", "3650d"]
        imagePullPolicy: IfNotPresent
---
```

## Start to verify that no mutual TLS - we CAN do curl commands

Let's now verify that there is no Mutual TLS that we could communicate among namespaces, pods and containers.

Begin by issuing a command from one namespace to another:
- **Source** = NS=bar, APP=sleep, PURPOSE=Try to reach destination NS=foo
- **Destination** = NS=foo, APP=httpbin, PURPOSE=Respond to source

You will need to issue a `kubectl exec` into the sleep container.

### Get the name of a pod

```
kubectl get pod -l app=sleep -n bar -o jsonpath={.items..metadata.name}
```

> sleep-7dc47f96b6-7dfld


 ### Get the containers in a pod

 
 ```
kubectl get pods sleep-7dc47f96b6-7dfld -n bar -o jsonpath='{.spec.containers[*].name}'
```

> 'sleep istio-proxy'


 ### Get information for Kubernetes Service INTERNAL endpoint
 
Now that we are in the `sleep` container, do a `curl`.

But before we issue the curl command, we need to target a specific container in the pod.

What is important now is to try to access the internal endpoint using the `curl` command.

The internal endpoint is composed of 3 pieces.

- Service Name
- Namespace
- Port

 ```
kubectl get services httpbin -o wide -n foo
```

Here is the output:

```
NAME      CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE       SELECTOR \
httpbin   10.0.103.141   <none>        8000/TCP   12h       app=httpbin
```

**Service Name** - httpbin

**Namespace** - foo

**Port** - 8000

**Result** - http://httbin.foo:8000


 ### Remote into a container that is in a specific pod and namespace
   
```
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar -- /bin/sh
```

####  Executing inside container - ready for command

```
Defaulting container name to sleep.
Use 'kubectl describe pod/sleep-7dc47f96b6-7dfld' to see all of the containers in this pod.
/ # 
```

```
/ curl http://httpbin.foo:8000 -w "%{http_code}\n"
```

### Curl command works! (`200` means `ok` as http code)

```
<h2 id="AUTHOR">AUTHOR</h2>
 
 </body>
 </html>200
```

## Turn Mutual TLS on and build DestinationRule

Let us begin by verifying there are no authentication policies.

```
kubectl get policies.authentication.istio.io --all-namespaces
```

Empty results:

```
No resources found.
```

Check  that there are no mesh policies.

```
kubectl get meshpolicies.authentication.istio.io
NAME      KIND
default   MeshPolicy.v1alpha1.authentication.istio.io

```


```
kubectl get destinationrules.networking.istio.io --all-namespaces -o yaml | grep "host:"
    host: istio-policy.istio-system.svc.cluster.local
    host: istio-telemetry.istio-system.svc.cluster.local
```
**No destination rules** - Verify that there are no destination rules that apply on the example services. 

**Check `host:` value** -  You can do this by checking the host: value of existing destination rules and make sure they do not match. For example:

```
kubectl get destinationrules.networking.istio.io --all-namespaces -o yaml

**RESULTS:**

apiVersion: v1
items:
- apiVersion: networking.istio.io/v1alpha3
  kind: DestinationRule
  metadata:
    creationTimestamp: 2019-01-30T04:46:58Z
    generation: 1
    name: istio-policy
    namespace: istio-system
    resourceVersion: "22419"
    selfLink: /apis/networking.istio.io/v1alpha3/namespaces/istio-system/destinationrules/istio-policy
    uid: 136ad272-244a-11e9-bdd7-8a8dfb8de0fa
  spec:
    host: istio-policy.istio-system.svc.cluster.local
    trafficPolicy:
      connectionPool:
        http:
          http2MaxRequests: 10000
          maxRequestsPerConnection: 10000
- apiVersion: networking.istio.io/v1alpha3
  kind: DestinationRule
  metadata:
    creationTimestamp: 2019-01-30T04:46:58Z
    generation: 1
    name: istio-telemetry
    namespace: istio-system
    resourceVersion: "22415"
    selfLink: /apis/networking.istio.io/v1alpha3/namespaces/istio-system/destinationrules/istio-telemetry
    uid: 135a6652-244a-11e9-bdd7-8a8dfb8de0fa
  spec:
    host: istio-telemetry.istio-system.svc.cluster.local
    trafficPolicy:
      connectionPool:
        http:
          http2MaxRequests: 10000
          maxRequestsPerConnection: 10000
kind: List
metadata: {}
resourceVersion: ""
selfLink: ""
```


## Globally enabling Mutual TLS

The next step is to submit a mass authentication policy. We will do this by creating a YAML files. The policy we create will specify that all workloads in the mesh will ONLY accept TLS encrypted requests. 

As you can see, this authentication policy has the kind: MeshPolicy. The name of the policy must be default, and it contains no targets specification (as it is intended to apply to all services in the mesh).

Notice the `kind:` as seen below:

#### default-mesh-policy.yaml

```
apiVersion: "authentication.istio.io/v1alpha1"
kind: "MeshPolicy"
metadata:
  name: "default"
spec:
  peers:
  - mtls: {}
EOF
```

### Require TLS for all communications

In this next test we are going to apply the rule that we just discussed, thereby preventing a `curl` command from succeeding from one container to another.

```
kubectl apply -f default-mesh-policy.yml

meshpolicy.authentication.istio.io/default configured
```

Get back inside the sleep service in ns=bar.

```
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar -- /bin/sh
```

Issue the command to the `httpbin` in the `foo` namespace.

```
/ curl http://httpbin.foo:8000 -w "%{http_code}\n"
```

At this point, only the receiving side is configured to use mutual TLS. If you run the curl command between Istio services (i.e those with sidecars), all requests will fail with a 503 error code as the client side is still using plain-text.

> upstream connect error or disconnect/reset before headers
> Error 503

## Configuring Destination rules

you can set up routing destination rules. using these rules you would be able to limit matches to only services in the cluster. This means that external services would not be able to communicate with services in the cluster.

These destination rules are also set up for non-authorization type of reasons. For example they can be used for `canarying.`

Allow internal traffic in cluster with Mutual TLS.

```
apiVersion: "networking.istio.io/v1alpha3"
kind: "DestinationRule"
metadata:
  name: "default"
  namespace: "default"
spec:
  host: "*.local"
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
```

Run the YAML file:

```
kubectl apply -f  all-internal-traffic.yml
destinationrule "default" created
```

Now we can see if we can once again hit an internal endpoint (httpbin from sleep).


 ### Remote into a container that is in a specific pod and namespace
   
```
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar -- /bin/sh
```

####  Executing inside container - ready for command

```
Defaulting container name to sleep.
Use 'kubectl describe pod/sleep-7dc47f96b6-7dfld' to see all of the containers in this pod.
/ # 
```

```
/ curl http://httpbin.foo:8000 -w "%{http_code}\n"
```

### Curl command works! (`200` means `ok` as http code)

```
<h2 id="AUTHOR">AUTHOR</h2>
 
 </body>
 </html>200
```

### Success! 

As you can see from the commands above, Re-running the testing allows all requests between Istio-services to be completed successfully.

## But should still should be off-limits to legacy apps

Those applications not running under the Istio fabric should be prevented from accessing cluster resources.



```
kubectl get pod -l app=sleep -n legacy -o jsonpath={.items..metadata.name}
```

 ### Remote into a container that is in a specific pod and namespace
   
```
kubectl exec -it sleep-7dc47f96b6-7dfld -n legacy -- /bin/sh
sleep-86cf99dfd6-ckndw
```

####  Executing inside container - ready for command

```
Defaulting container name to sleep.
Use 'kubectl describe pod/sleep-86cf99dfd6-ckndw' to see all of the containers in this pod.
/ # 
```

```
/ curl http://httpbin.foo:8000 -w "%{http_code}\n"
```

### Curl command works! (`200` means `ok` as http code)

```
(6) Could not resolve host: httbin.foo 000
curl: (56) Recv failure: Connection reset by peer
```

### Success again!

so the results above are in alignment with our expectations. We do support service to service communication, as long as it is within the service mesh/Istio environment. The `legacy` namespace, however, was created outside the service mesh/Istio environment. Therefore, any applications from inside the `legacy` namespace are unable to communicate with any of the services that run within the service mesh. 



