## Demo Script

#### Demo List

* Automatic proxy injection​
* Observability – Grafana dashboard views​
* Ingress Gateway
* Dynamic traffic routing​
* Distributed Tracing​
* Canary deployments with Flagger 

#### Automatic proxy injection​

1. Start with an empty cluster with Istio installed
    ```bash
    kubectl get pod -n aks-istio-system
    kubectl get pod -n aks-istio-ingress
    ```

2. Explain `istioctl kube-inject --help` and the "mutating webhook admission controller"
    ```bash
    kubectl describe MutatingWebhookConfiguration istio-sidecar-injector-asm-1-17-aks-istio-system
    ```
3. Create a ns
4. Label the ns and show
5. Deploy app via YAML and show sidecars (describe order-service pod)

#### Observability – Grafana dashboard views​

1. Run a load test on the application (night before)
2. Switch to cluster with previously deployed app
3. Show control plane components and explain how to setup Prometheus integration
4. Show Grafana dashboard
5. Show some interesting views including the load test

#### Ingress Gateway

1. Show ingress gateway pods in `aks-istio-ingress` namespace
2. Show how we get the IP address and port for the gateway
3. Show nslookup on *.brianredmond.io 
4. Show ingress YAML manifests (Gateway, Virtual Service)
5. Deploy the manifests
    * http://petstore.brianredmond.io/
    * http://petstore-admin.brianredmond.io/

#### Dynamic traffic routing​

1. Show the order-service deployments (both versions) 
2. Show traffic is split randomly
3. Deploy Istio virtual service, rules (all v2)
4. Shift traffic to v3 canary version


