# Installation

```
kubectl apply -f https://raw.githubusercontent.com/kinvolk/headlamp/main/kubernetes-headlamp.yaml
```

# Usage

kubectl -n kube-system create serviceaccount headlamp-admin
kubectl create clusterrolebinding headlamp-admin --serviceaccount=kube-system:headlamp-admin --clusterrole=cluster-admin


# Find the secret that was created to hold the token for the SA
kubectl get secrets

# Show the contents of the secret to extract the token
kubectl describe secret skooner-sa-token-xxxxx
kubectl create token skooner-sa -n kube-system

```bash
export INGRESS_IP=$(kubectl get service --namespace ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[].ip}')
echo $INGRESS_IP
export INGRESS_DOMAIN=${INGRESS_IP}.nip.io
echo $INGRESS_DOMAIN
```

```bash

envsubst < headlamp_ingress.yaml | kubectl apply -f -
kubectl get ing -A

```