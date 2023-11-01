## Traffic Shifting

Docs: https://istio.io/latest/docs/tasks/traffic-management/traffic-shifting
Book Info all: https://github.com/istio/istio/tree/master/samples/bookinfo/platform/kube
Book Info base deploy: https://github.com/istio/istio/blob/master/samples/bookinfo/platform/kube/bookinfo.yaml 
Book Info destination rules: https://raw.githubusercontent.com/istio/istio/release-1.19/samples/bookinfo/networking/destination-rule-all.yaml

#### Initial setup

```bash
kubectl apply -f ./local-files/order-service-new.yaml -n pets

kubectl run testing-pod --image=nginx -n pets
kubectl exec -it testing-pod -n pets -- bash
export APP_URL=http://order-service.pets:3000/health
while true; do curl -s -w "\n" $APP_URL; sleep 1; done
```

#### Add traffic shifting rule

```bash
kubectl apply -f ./local-files/order-svc-destination-rule.yaml -n pets

# route all traffice to v2
kubectl apply -f ./local-files/order-svc-all-v2.yaml -n pets

# split 90% canary traffic to v3 
kubectl apply -f ./local-files/order-svc-split.yaml -n pets
```


#### Clean-up

```bash
kubectl delete -f ./local-files/order-service-new.yaml -n pets
kubectl delete pod testing-pod -n pets
```




