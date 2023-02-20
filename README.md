# Demo-Istio

## Summary

Istio Demo based on `istio-1.17.0`. This demo is only for cluster with `external-LB`.

## Pre steps

Move into the Istio package directory:
```
cd istio-1.17.0
```

Add the istioctl client to your path (Linux or macOS):
```
export PATH=$PWD/bin:$PATH
```

## Install Istio

We use the demo configuration profile:
```
istioctl install --set profile=demo -y
```

Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:
```
kubectl label namespace default istio-injection=enabled
```

## Deploy the sample application

Deploy the Bookinfo sample application:
```
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

Check the app service, namespace and pods:
```
kubectl get ns
```
Check the istio-ingressgateway status with:
```
kubectl get svc -n istio-system

NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP          PORT(S)                                                                      AGE
istio-egressgateway    ClusterIP      10.108.33.143    <none>               80/TCP,443/TCP                                                               4m34s
istio-ingressgateway   LoadBalancer   10.101.160.175   <YOUR-IP-ADDRESS>    15021:20278/TCP,80:16232/TCP,443:30867/TCP,31400:12614/TCP,15443:24043/TCP   4m34s
istiod                 ClusterIP      10.100.152.218   <none>               15010/TCP,15012/TCP,443/TCP,15014/TCP                                        5m3s
```

then run the following checks:
```
kubectl get deploy
kubectl get svc
kubectl get pods
```

Verify everything is working correctly up to this point. Run this command to see if the app is running inside the cluster and serving HTML pages by checking for the page title in the response:
```
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

## Open the application to outside traffic

The Bookinfo application is deployed but not accessible from the outside. To make it accessible, you need to create an Istio Ingress Gateway, which maps a path to a route at the edge of your mesh.

Associate this application with the Istio gateway:
```
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```
Ensure that there are no issues with the configuration:
```
istioctl analyze
```
Output:
```
✔ No validation issues found when analyzing namespace: default.
```

Apps is listening on:
```
http://<YOUR-EXT£ERNAL-IP/productpage
```

## View the dashboard

Install Kiali and the other addons and wait for them to be deployed.
```
kubectl apply -f samples/addons

serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```

Check deployment status with:
```
kubectl rollout status deployment/kiali -n istio-system

Waiting for deployment "kiali" rollout to finish: 0 of 1 updated replicas are available...
deployment "kiali" successfully rolled out
```

tehn check dashboard svc:
```
kubectl get svc -n istio-system

NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP          PORT(S)                                                                      AGE
grafana                ClusterIP      10.108.41.238    <none>               3000/TCP                                                                     3m49s
istio-egressgateway    ClusterIP      10.108.33.143    <none>               80/TCP,443/TCP                                                               108m
istio-ingressgateway   LoadBalancer   10.101.160.175   <YOUR-IP-ADDRESS>    15021:20278/TCP,80:16232/TCP,443:30867/TCP,31400:12614/TCP,15443:24043/TCP   108m
istiod                 ClusterIP      10.100.152.218   <none>               15010/TCP,15012/TCP,443/TCP,15014/TCP                                        109m
jaeger-collector       ClusterIP      10.109.109.48    <none>               14268/TCP,14250/TCP,9411/TCP                                                 3m48s
kiali                  ClusterIP      10.111.85.65     <none>               20001/TCP,9090/TCP                                                           3m48s
prometheus             ClusterIP      10.101.151.180   <none>               9090/TCP                                                                     3m48s
tracing                ClusterIP      10.109.10.62     <none>               80/TCP,16685/TCP                                                             3m49s
zipkin                 ClusterIP      10.96.236.34     <none>               9411/TCP                                                                     3m49s
```

Change `ClusterIP` to `LoadBalancer`:
```
kubectl edit svc kiali -n istio-system
...
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer: {}
```

After save and exit:
```
kubectl get svc -n istio-system

NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP          PORT(S)                                                                      AGE
grafana                ClusterIP      10.108.41.238    <none>               3000/TCP                                                                     7m20s
istio-egressgateway    ClusterIP      10.108.33.143    <none>               80/TCP,443/TCP                                                               112m
istio-ingressgateway   LoadBalancer   10.101.160.175   <YOUR-IP-ADDRESS>    15021:20278/TCP,80:16232/TCP,443:30867/TCP,31400:12614/TCP,15443:24043/TCP   112m
istiod                 ClusterIP      10.100.152.218   <none>               15010/TCP,15012/TCP,443/TCP,15014/TCP                                        112m
jaeger-collector       ClusterIP      10.109.109.48    <none>               14268/TCP,14250/TCP,9411/TCP                                                 7m19s
kiali                  LoadBalancer   10.111.85.65     <YOUR-IP-ADDRESS>    20001:19754/TCP,9090:27676/TCP                                               7m19s
prometheus             ClusterIP      10.101.151.180   <none>               9090/TCP                                                                     7m19s
tracing                ClusterIP      10.109.10.62     <none>               80/TCP,16685/TCP                                                             7m20s
zipkin                 ClusterIP      10.96.236.34     <none>               9411/TCP                                                                     7m20s
```

then the dashboard available on:
```
http://<YOUR-IP-ADDRESS>:20001/kiali
```

## Uninstall

Steps are:
```
kubectl delete -f samples/addons
istioctl uninstall -y --purge
kubectl delete namespace istio-system
kubectl label namespace default istio-injection-
```