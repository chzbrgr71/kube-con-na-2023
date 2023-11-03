## Install App

#### Deploy application

https://istio.io/latest/docs/setup/additional-setup/sidecar-injection 

```bash
kubectl create ns pets
kubectl label namespace pets istio.io/rev=asm-1-17
kubectl get namespace -L istio.io/rev

kubectl apply -f ./local-files/aks-store-all-in-one.yaml -n pets
kubectl get pod -n pets
```

#### Configure Application Ingress

https://learn.microsoft.com/en-us/azure/aks/istio-deploy-ingress

```bash
# external
kubectl apply -f ./manifests/ingress-store-front.yaml
kubectl apply -f ./manifests/ingress-store-admin.yaml

export INGRESS_HOST_EXTERNAL=$(kubectl -n aks-istio-ingress get service aks-istio-ingressgateway-external -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT_EXTERNAL=$(kubectl -n aks-istio-ingress get service aks-istio-ingressgateway-external -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export GATEWAY_URL_EXTERNAL=$INGRESS_HOST_EXTERNAL:$INGRESS_PORT_EXTERNAL
echo "http://$GATEWAY_URL_EXTERNAL"

http://petstore.brianredmond.io/
http://petstore-admin.brianredmond.io/

kubectl delete -f ./manifests/ingress-store-front.yaml
kubectl delete -f ./manifests/ingress-store-admin.yaml

# internal
kubectl apply -f ./manifests/ingress-internal-order-service.yaml

export INGRESS_HOST_INTERNAL=$(kubectl -n aks-istio-ingress get service aks-istio-ingressgateway-internal -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT_INTERNAL=$(kubectl -n aks-istio-ingress get service aks-istio-ingressgateway-internal -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export GATEWAY_URL_INTERNAL=$INGRESS_HOST_INTERNAL:$INGRESS_PORT_INTERNAL
echo "http://$GATEWAY_URL_INTERNAL"

kubectl run mycurlpod --image=curlimages/curl --restart=Never --rm -it -q -- curl "http://$GATEWAY_URL_INTERNAL/health"

while true; do kubectl run mycurlpod --image=curlimages/curl --restart=Never --rm -it -q -- curl "http://$GATEWAY_URL_INTERNAL/health"; sleep 1; done

kubectl run nginx --image=nginx
export APP_URL=http://192.168.1.1/health
while true; do curl -s -w "\n" $APP_URL; sleep 1; done
```

#### Uninstall application

```bash
kubectl delete -f .. -n pets
```

#### New Release

```bash
gh auth login --scopes repo,workflow,write:packages

gh repo fork https://github.com/azure-samples/aks-store-demo.git --clone

cd aks-store-demo

gh repo set-default

gh release create 1.0.0 --generate-notes

gh run watch

```



