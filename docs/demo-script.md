## Demo Script

#### Demo List

* Automatic proxy injection​
* Ingress Gateway
* Observability – Grafana dashboard views​
* Dynamic traffic routing​

* Canary deployments with Flagger

https://docs.flagger.app/install/flagger-install-on-kubernetes
https://paulyu.dev/article/more-gitops-with-fluxcd-and-flagger-on-aks
https://istio.io/latest/docs/ops/integrations/prometheus 

#### Automatic proxy injection​

0. Start with an empty cluster [service-mesh-01]
    * Istio Add-on 
    * Prometheus/Grafana
    * No ingress gateways

1. Show Istio control plane
    ```bash
    kubectl get pod -n aks-istio-system
    ```

2. Explain `istioctl kube-inject --help` and the "mutating webhook admission controller"
    ```bash
    kubectl describe MutatingWebhookConfiguration istio-sidecar-injector-asm-1-17-aks-istio-system
    ```

3. Create a ns and label
    ```bash
    kubectl create ns pets
    kubectl label namespace pets istio.io/rev=asm-1-17
    kubectl get namespace -L istio.io/rev
    ```

4. Deploy app via YAML and show sidecars
    ```bash
    kubectl apply -f ./local-files/aks-store-all-in-one.yaml -n pets
    kubectl get pod -n pets

    kubectl describe pod order-service-v2-cd78f884f-xvbnw -n pets
    ```
5. Show port-forward to store front
    ```bash
    kubectl get pod -n pets
    kubectl port-forward -n pets store-front-6d5469bb97-2ckns 8080:8080
    ```

#### Ingress Gateway

0. Switch to pre-deployed cluster [service-mesh-02]
    * Prometheus/Grafana setup
    * Istio Gateways installed
    * Pre-deploy store app with both order-service versions
    * No ingress setup for store-front
    * Testing pod running

1. Show ingress gateway pods in `aks-istio-ingress` namespace
    ```bash
    kubectl get pod -n aks-istio-ingress
    ```

2. Show how we get the IP address and port for the gateway
    ```bash
    export INGRESS_HOST_EXTERNAL=$(kubectl -n aks-istio-ingress get service aks-istio-ingressgateway-external -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    export INGRESS_PORT_EXTERNAL=$(kubectl -n aks-istio-ingress get service aks-istio-ingressgateway-external -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    export GATEWAY_URL_EXTERNAL=$INGRESS_HOST_EXTERNAL:$INGRESS_PORT_EXTERNAL
    echo $INGRESS_HOST_EXTERNAL
    ```

3. Show DNS
    ```bash
    nslookup pets.brianredmond.io
    ```

4. Show ingress YAML manifests (Gateway, Virtual Service)
    * `ingress-store-front.yaml`

5. Deploy the manifests
    ```bash
    kubectl apply -f ./manifests/ingress-store-front.yaml
    ```

6. Show web app
    * http://pets.brianredmond.io/

#### Observability – Grafana dashboard views​

0. Start with same cluster as previous demo [service-mesh-02]
    * Run a load test on the application (night before)
    * `kubectl scale --replicas=5 deployment.apps/virtual-customer -n pets`

1. Show docs and explain. https://istio.io/latest/docs/ops/integrations/prometheus 

2. Show Grafana dashboard views

3. Show results of Load Test

#### Dynamic traffic routing​

0. Start with same cluster as gateway demo [service-mesh-02]
1. Show the order-service deployments (both versions) & configs
2. Show traffic is split randomly
    ```bash
    kubectl run testing-pod --image=nginx -n pets
    kubectl exec -it testing-pod -n pets -- bash

    export APP_URL=http://order-service.pets:3000/health
    while true; do curl -s -w "\n" $APP_URL; sleep 1; done
    ```

3. Deploy Istio virtual service, rules (all v1)
    ```bash
    kubectl apply -f ./manifests/order-svc-destination-rule.yaml
    kubectl apply -f ./manifests/order-svc-all-v1.yaml
    ```

4. Shift traffic to v3 canary version
    ```bash
    # set to 75/25
    kubectl apply -f ./manifests/order-svc-split.yaml

    # reverse the ratio
    kubectl apply -f ./manifests/order-svc-split.yaml
    ```

    ```bash
    # remove rules
    kubectl delete -f ./manifests/order-svc-destination-rule.yaml
    kubectl delete -f ./manifests/order-svc-split.yaml
    ```


#### Flagger (maybe)

0. Basic AKS cluster with no add-ons [service-mesh-03]
    * Install OSS Istio (istioctl)
    * Deploy Prometheus https://istio.io/latest/docs/ops/integrations/prometheus 
    * Depoy Grafana https://istio.io/latest/docs/ops/integrations/grafana
    * Deploy app and v2 order-service
    * Setup Flagger

