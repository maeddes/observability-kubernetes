# Install

```bash

curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.17.1 sh -
istio-1.17.1/bin/istioctl install --set profile=default -y 
kubectl apply -f istio-1.17.1/samples/addons

```

# After installation

show k9s live deployment or do

```bash

kubectl get all -n istio-system

kubectl get service --namespace istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[].ip}'

INGRESS_IP_ISTIO=$(kubectl get service --namespace istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[].ip}'); echo Istio Ingress: $INGRESS_IP_ISTIO

export INGRESS_DOMAIN=${INGRESS_IP_ISTIO}.nip.io; echo $INGRESS_DOMAIN
        
```

# Configure Ingress Gateway

Replace all instances of ${INGRESS_DOMAIN} with actual IP (in VSCode)

```bash

envsubst < istio-ingress.yaml | kubectl apply -f -
kubectl get gateway,virtualservice,destinationrule -A

```

# Configure injection of single pod

```
kubectl get deployments todoui -n todoapp -o yaml 
kubectl get deployments todoui -n todoapp -o yaml | istioctl kube-inject -f - 
kubectl get deployments todoui -n todoapp -o yaml | istioctl kube-inject -f - | kubectl apply -f -

kubectl get pod -n todoapp -w
```

# Show in Kiali


# Configure injection of all pods

```
kubectl get deployments -n todoapp -o yaml | istioctl kube-inject -f - | kubectl apply -f -
```

# Generate load (via URL :-))

```
export INGRESS_IP=$(kubectl get service --namespace ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[].ip}')
echo $INGRESS_IP
export INGRESS_DOMAIN=todoui.${INGRESS_IP}.nip.io; echo $INGRESS_DOMAIN

CURL="curl  --location"
TODO='NEW_TODO'
while sleep 1; do
    $CURL http://$INGRESS_DOMAIN/      --data toDo=$TODO    # i.e. POST for creating an entry followed by GET for listing
    echo http://$INGRESS_DOMAIN/      --data toDo=$TODO
    sleep 1
    $CURL http://$INGRESS_DOMAIN/$TODO --data value='Done!' # i.e. POST for deleting an entry followed by GET for listing
    date
done # > /dev/null
```

# Make it nice

```bash
kubectl patch service -n todoapp postgresdb  --patch='{ "spec": { "ports": [ {"port": 5432, "name": "tcp-postgres" } ] } }'
kubectl patch service -n todoapp todobackend --patch='{ "spec": { "ports": [ {"port": 8080, "name": "http-todobackend" } ] } }'
kubectl patch service -n todoapp todoui      --patch='{ "spec": { "ports": [ {"port": 8090, "name": "http-todoui" } ] } }'
for i in postgresdb todobackend todoui; do kubectl label -n todoapp service $i app=$i; done
```

# Instrument an entire namespace

Petclinic
```
kubectl label namespace spring-petclinic istio-injection=enabled --overwrite
kubectl get pod -n spring-petclinic -o yaml  | kubectl delete -f -
```

Otel
```
kubectl label namespace otel istio-injection=enabled --overwrite
kubectl get pod -n otel -o yaml  | kubectl delete -f -
```

Removal
```
kubectl label namespace otel istio-injection=disabled --overwrite
kubectl get pod -n otel -o yaml  | kubectl delete -f -
```

# Prometheus and Grafana



http://grafana.20.76.85.123.nip.io/d/UbsSZTDik/istio-workload-dashboard?orgId=1&var-datasource=default&var-namespace=todoapp&var-workload=todobackend&var-qrep=destination&var-srcns=All&var-srcwl=All&var-dstsvc=All&from=now-3h&to=now&refresh=1m