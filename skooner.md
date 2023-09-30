# Installation

```
kubectl apply -f 
https://raw.githubusercontent.com/skooner-k8s/skooner/master/kubernetes-skooner.yaml
```

# Usage

kubectl create serviceaccount skooner-sa
kubectl create clusterrolebinding skooner-sa --clusterrole=cluster-admin --serviceaccount=default:skooner-sa

# Find the secret that was created to hold the token for the SA
kubectl get secrets

# Show the contents of the secret to extract the token
kubectl describe secret skooner-sa-token-xxxxx