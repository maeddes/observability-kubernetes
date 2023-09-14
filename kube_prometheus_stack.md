```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack
```

```
kubectl port-forward svc/prometheus-grafana 8080:80
```

admin:prom-operator

http://localhost:8080/dashboards
