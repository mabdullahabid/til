# Useful Azure Kubernetes Service (AKS) commands

Make sure you have the Azure CLI installed and logged in.

## Get AKS credentials
```bash
az account set --subscription <subscription-id>
az aks get-credentials --resource-group <resource-group> --name <cluster-name> --overwrite-existing
```

## List AKS pods
```bash
kubectl get pods --namespace <namespace>
```
