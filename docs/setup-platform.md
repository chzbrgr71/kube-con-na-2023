## Setup Platform

#### Monitoring

```bash
export RG_MONITORING=service-mesh-monitoring
export LOCATION=centralus
export WORKSPACENAME=btr-azuremonitor-ws-02
export GRAFANANAME=btr-grafana-02

az group create -n $RG_MONITORING -l $LOCATION

az resource create --resource-group $RG_MONITORING --namespace microsoft.monitor --resource-type accounts --name $WORKSPACENAME --location $LOCATION --properties "{}"

az grafana create --name $GRAFANANAME --resource-group $RG_MONITORING
```

#### AKS w/ Istio Add-on

```bash
export RG=service-mesh-02
export RG_MONITORING=service-mesh-monitoring
export LOCATION=centralus
export CLUSTERNAME=service-mesh-02
export K8SVERSION=1.27.1
export NODECOUNT=5
export VMSIZE=Standard_DS3_v2
export WORKSPACENAME=btr-azuremonitor-ws-02
export GRAFANANAME=btr-grafana-02

az group create -n $RG -l $LOCATION

WORKSPACEID=$(az monitor account show -n $WORKSPACENAME -g $RG_MONITORING --query id -o tsv)
echo $WORKSPACEID

GRAFANAID=$(az grafana show -n $GRAFANANAME -g $RG_MONITORING --query id -o tsv)
echo $GRAFANAID

az aks create \
    -g $RG \
    -n $CLUSTERNAME \
    --node-vm-size $VMSIZE \
    --kubernetes-version $K8SVERSION \
    --tier standard \
    --enable-azure-monitor-metrics \
    --azure-monitor-workspace-resource-id $WORKSPACEID \
    --grafana-resource-id $GRAFANAID \
    --enable-asm \
    --node-count $NODECOUNT

az aks get-credentials -n $CLUSTERNAME -g $RG
```

#### Configure Istio for Prometheus/Grafana

```bash
kubectl apply -f ./manifests/ama-metrics-prom.yaml
```

Add Dashboards to Grafana
https://grafana.com/grafana/dashboards/?search=istio
https://grafana.com/grafana/dashboards/7630-istio-workload-dashboard

#### Deploy Istio Ingress Gateway

https://learn.microsoft.com/en-us/azure/aks/istio-deploy-ingress

```bash
az aks mesh enable-ingress-gateway --resource-group $RG --name $CLUSTERNAME --ingress-gateway-type external

az aks mesh enable-ingress-gateway --resource-group $RG --name $CLUSTERNAME --ingress-gateway-type internal

kubectl get svc -n aks-istio-ingress
```

#### AKS Basic Install

```bash
export RG=service-mesh-03
export LOCATION=centralus
export CLUSTERNAME=service-mesh-03
export K8SVERSION=1.27
export NODECOUNT=5
export VMSIZE=Standard_DS3_v2

az group create -n $RG -l $LOCATION

az aks create \
    -g $RG \
    -n $CLUSTERNAME \
    --node-vm-size $VMSIZE \
    --kubernetes-version $K8SVERSION \
    --tier standard \
    --node-count $NODECOUNT

az aks get-credentials -n $CLUSTERNAME -g $RG

# install Istio
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

kubectl port-forward -n pets store-front-6d5469bb97-559r9 8080:8080
```



