# Installation

```bash
kubectl apply -f https://raw.githubusercontent.com/kinvolk/headlamp/main/kubernetes-headlamp.yaml
```

# Usage

```bash
kubectl -n kube-system create serviceaccount headlamp-admin
kubectl create clusterrolebinding headlamp-admin --serviceaccount=kube-system:headlamp-admin --clusterrole=cluster-admin
```

# Show the contents of the secret to extract the token
```bash
kubectl create token headlamp-admin -n kube-system
```

```bash
export INGRESS_IP=$(kubectl get service --namespace ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[].ip}')
echo $INGRESS_IP
export INGRESS_DOMAIN=${INGRESS_IP}.nip.io
echo $INGRESS_DOMAIN
```

```bash
kubectl create ingress headlamp -n kube-system --class=nginx --rule="headlamp.${INGRESS_DOMAIN}/*=headlamp:80"
```

```bash
envsubst < headlamp_ingress.yaml | kubectl apply -f -
kubectl get ing -A
```
