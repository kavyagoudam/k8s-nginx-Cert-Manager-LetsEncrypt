# AKS Nginx Cert manager and Let's encyrpt

In this project, We will achieve the following
1.  Pre-requisites
2.  Nginx controller setup
3.  Cert manager configuration
4.  Application deployment with service 
5.  ClusterIssuer creation
6.  certificate creation
7.  edit ingress with certification

1. Pre-requirements
  a. Kubernetes Cluster with admin access. if not you can create AKS in Microsoft Azure, using below command.

```cli
AKS_RG="rg-cert-demo"
AKS_NAME="aks-cert-demo"
az group create -n $AKS_RG -l eastus
az aks create -g $AKS_RG -n $AKS_NAME --kubernetes-version "1.29.8" --node-count 3 --network-plugin azure
az aks get-credentials -n $AKS_NAME -g $AKS_RG --overwrite-existing
```
  b. clone the repo, and move inside the directory
     

2.  Nginx Controller setup
    Install the Nginx Controller
    ```
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/cloud/deploy.yaml
    kubectl get pods --namespace=ingress-nginx
    NAME                                        READY   STATUS      RESTARTS   AGE
    ingress-nginx-admission-create-4j2sh        0/1     Completed   0          152m
    ingress-nginx-admission-patch-hlfwm         0/1     Completed   0          152m
    ingress-nginx-controller-66f986c956-4kwc9   1/1     Running     0          152m
    ```
4.  Cert manager configuration
    ```
    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.1/cert-manager.yaml
    kubectl get pods --namespace cert-manager
    ```
5.  Application deployment with service
    Deploy the Rancher application using the below command
    ```
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```
6.  create the cluster issue
    ```
    kubectl apply -f cluster-issuer-production.yaml
    kubectl apply -f cluster-issuer-staging.yaml
    ```
7.  edit ingress with certification
    
    


   
