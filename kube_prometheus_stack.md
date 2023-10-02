```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

```
helm install prometheus prometheus-community/kube-prometheus-stack -n kube-prometheus-stack --create-namespace
```

```bash
export INGRESS_IP=$(kubectl get service --namespace ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[].ip}')
echo $INGRESS_IP
export INGRESS_DOMAIN=${INGRESS_IP}.nip.io
echo $INGRESS_DOMAIN
```

```bash

envsubst < kube_prometheus_ingress.yaml | kubectl apply -f -
kubectl get ing -A

```
admin:prom-operator


