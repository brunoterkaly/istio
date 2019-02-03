
# Misc Commands 

Sommand commands I'm collecting.

```bash

FORTIO_POD=$(kubectl get pod | grep fortio | awk "{ print $1 }")
history | grep "kubectl.*istioctl"
history | grep "kubectl.*istioctl" > test.txt & vim test.txt
history | grep "kubectl.*istioctl" > test.txt && vim test.txt
kubectl apply -f  all-internal-traffic.yml
kubectl apply -f circuit-breaker.yml
kubectl apply -f default-mesh-policy.yml
kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n bar
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/sample-client/fortio-deploy.yaml)
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n bar
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo
kubectl apply -f kubernetes/step-1-create-voting-app.yaml --namespace voting
kubectl apply -f kubernetes/step-2-update-voting-analytics-to-1.1.yaml --namespace voting
kubectl apply -f kubernetes/step-3-update-voting-app-with-new-storage.yaml --namespace voting
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
kubectl apply -f samples/httpbin/httpbin.yaml -n legacy
kubectl apply -f < samples/sleep/sleep.yaml -n legacy
kubectl apply -f samples/sleep/sleep.yaml -n legacy
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
kubectl create -f < (istioctl kube-inject -f bookinfo.yaml)
kubectl create -f <(istioctl kube-inject -f bookinfo.yaml)
kubectl create -f <(istioctl kube-inject -f $(BOOKINFOYAML))
kubectl create -f <(istioctl kube-inject -f ${BOOKINFOYAML})
kubectl create namespace voting
kubectl create ns bar
kubectl create ns foo
kubectl create ns legacy
kubectl delete deploy httpbin fortio-deploy
kubectl delete destinationrule httpbin
kubectl delete -f circuit-breaker.yml
kubectl delete svc httpbin
kubectl describe pod voting-app-1-0-6c65c4bdd4-bdmld --namespace voting
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
kubectl exec -it $FORTIO_POD  -c fortio /usr/local/bin/fortio -- load -curl  http://httpbin:8000/get
kubectl exec -it $FORTIO_POD  -c istio-proxy  -- sh -c "curl localhost:15000/stats" | grep httpbin | grep pending
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar -- /bin/sh
- kubectl exec -it sleep-7dc47f96b6-7dfld -n bar --container sleep -- /bin/sh
kubectl exec -it sleep-7dc47f96b6-7dfld -n bar --container sleep -- /bin/sh
kubectl exec -it sleep-7dc47f96b6-7dfld -n legacy -- /bin/sh
kubectl get destinationrule httpbin -o yaml
kubectl get destinationrules.networking.istio.io --all-namespaces -o yaml
kubectl get destinationrules.networking.istio.io --all-namespaces -o yaml | grep "host:"
kubectl get destinationrules  #-- there should be no destination rules
kubectl get gateway
kubectl get gateway           #-- there should be no gateway
kubectl get meshpolicies.authentication.istio.io
kubectl get pod -l app=sleep -n bar -o jsonpath= {.items..metadata.name}
kubectl get pod -l app=sleep -n bar -o jsonpath={.items..metadata.name}
kubectl get pod -l app=sleep -n legacy -o jsonpath={.items..metadata.name}
kubectl get pods
kubectl get pods --all-namespaces
kubectl get pods --namespace voting
kubectl get pods -n istio-system
kubectl get pods -n voting
kubectl get pods sleep-7dc47f96b6-7dfld -n bar -o jsonpath="{.spec.containers[*].name}"
kubectl get pods               #-- the Bookinfo pods should be deleted
kubectl get policies.authentication.istio.io --all-namespaces
kubectl get service istio-ingressgateway --namespace istio-system -o jsonpath="{.status.loadBalancer.ingress[0].ip}"
kubectl get services
kubectl get services httpbin -o wide -n foo
kubectl get svc istio-ingressgateway -n istio-system
kubectl get virtualservices   #-- there should be no virtual services
kubectl.kubernetes.io/last-applied-configuration: |
kubectl label namespace voting istio-injection=enabled
kubectl proxy




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
kubectl get pods sleep-7dc47f96b6-7dfld -n bar -o jsonpath="{.spec.containers[*].name}"
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
kubectl get pods sleep-7dc47f96b6-7dfld -n bar -o jsonpath="{.spec.containers[*].name}"
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
helm ls | awk "{print $1}"
helm ls | awk "{print $1}" | vim -
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
tar xvzf helm-v2.12.1-linux-amd64.tar.gz
```

