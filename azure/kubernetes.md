# Useful Azure Kubernetes Service (AKS) 

A collection of useful commands and tips for working with Azure Kubernetes Service (AKS).

Make sure you have the Azure CLI installed and logged in.

## Get AKS credentials

    az account set --subscription <subscription-id>
    az aks get-credentials --resource-group <resource-group> --name <cluster-name> --overwrite-existing


## List AKS pods

    kubectl get pods --namespace <namespace>

### Optional: Lens IDE

Download Lens from https://k8slens.dev/

Lens is an IDE built specifically for Kubernetes that lets you connect to clusters and view, monitor and maintain your clusters.

### Get contexts

    kubectl config get-contexts

### Restart all deployments

    kubectl rollout restart deployment -n <namespace>