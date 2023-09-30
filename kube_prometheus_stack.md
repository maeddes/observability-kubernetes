```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack
```

```bash

envsubst < kube_prometheus_ingress.yaml | kubectl apply -f -
kubectl get ing -A

```
admin:prom-operator


