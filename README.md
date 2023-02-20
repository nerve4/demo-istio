# Demo-Istio

## Summary

Istio Demo based on `istio-1.17.0`.

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