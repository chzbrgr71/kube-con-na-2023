## Istio OSS Setup

#### Manual install of OSS Istio

```bash
istioctl install --set profile=demo -y
```

#### Deploy application

https://istio.io/latest/docs/setup/additional-setup/sidecar-injection 

```bash
kubectl create ns pets
kubectl label namespace pets istio-injection=enabled --overwrite
kubectl get namespace -L istio-injection

kubectl apply -f ./local-files/aks-store-all-in-one.yaml -n pets
kubectl get pod -n pets

kubectl port-forward -n pets store-front-6d5469bb97-2ckns 8080:8080
```