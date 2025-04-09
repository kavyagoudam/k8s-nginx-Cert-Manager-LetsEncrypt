# Kubernetes Nginx Cert manager and Let's encyrpt

## Enable HTTPS for website running in kubernetes cluster
TLS(Transport Layer Security) protocol add security layer on top of  http connection. TLS protocol wrap around the http connection, So that the data which exchanged between server and client is secured. To enable the HTTPS connection, server needs to have certificate.

## Certificate
Non trust certificate will give the warning in browser saying that connection is not private, otherwise you will see the lock on top left corner of the browser.

In this tutorial, We will install nginx ingress controller, Cert Manager, obtain a valid certificate for our application by using non-profit Certificate Authority(CA) called Let's Encrypt.
Nginx controller is used as LoadBalancer between the application and the Cert Manager is used to communicate with Let's Encrypt. Let's Encrypt will give Valid certificate to your domain.

1. Pre-requisites and preparation
  a. Kubernetes Cluster with admin access. if not you can create AKS in Microsoft Azure, using below command.

```cli
AKS_RG="rg-cert-demo"
AKS_NAME="aks-cert-demo"
az group create -n $AKS_RG -l eastus
az aks create -g $AKS_RG -n $AKS_NAME --kubernetes-version "1.29.8" --node-count 3 --network-plugin azure
az aks get-credentials -n $AKS_NAME -g $AKS_RG --overwrite-existing
```

2.  Nginx Controller and Cert Manager configuration
   a. Install the Nginx Controller
    ```
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/cloud/deploy.yaml
    kubectl get pods --namespace=ingress-nginx
    NAME                                        READY   STATUS      RESTARTS   AGE
    ingress-nginx-admission-create-4j2sh        0/1     Completed   0          152m
    ingress-nginx-admission-patch-hlfwm         0/1     Completed   0          152m
    ingress-nginx-controller-66f986c956-4kwc9   1/1     Running     0          152m
    ```
    b.  Cert manager configuration
    ```
    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.1/cert-manager.yaml
    kubectl get pods --namespace cert-manager
    ```
3.  Application deployment with service
    If you have a demo application, bring that application into kubernetes, otherwise you can use this repo to build the repo.
    
    a. clone the repo, and move inside the directory

    ```
    git clone https://github.com/kavyagoudam/k8s-nginx-Cert-Manager-LetsEncrypt.git
    cd k8s-nginx-Cert-Manager-LetsEncrypt/
    ```
    b. Deploy the Rancher application using the below command

    ```
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```
    Once the application is deployed we need to apply ingress. Take a look at ingress.yaml. replace <YOUR_IP_ADDRESS> with Ingress LoadBalancer ip address. If you have valid domain name you can give your domain name as well.
    .nib.io domain will route the traffic to IP address as subdomain. 
    
    ```
    kubectl apply -f ingress.yaml
    ```
    once the application is created, you can access the webservice from the browser https://<your_IP_ADDRESS>.nip.io
    
    ![image](https://github.com/user-attachments/assets/0230dd15-0f46-421c-9ca6-bde7a0d86d7c)

    in browser you can see that connection is not secure.
    
    click on "Not Secure" in browser, and the certificate warning. you can see that it's a Fake certificate.
    ![image](https://github.com/user-attachments/assets/78634120-19ad-4980-8516-c1e8b923f44b)
    The Goal of this tutorial is to change this to valid certificate.
    
4.  create the cluster issue
    In the previuos step, we have installed the cert manager, now we will configure cert-manager to communicate with Let's encrypt using Cluster issuer.
    replace the <your_email_address>  with actual email addres.
    ```
    kubectl apply -f cluster-issuer-production.yaml
    kubectl apply -f cluster-issuer-staging.yaml
    ```
    In cluster-issuer-production.yaml and cluster-issuer-staging.yaml,
    
    kind:ClusterIssuer -> it depends on the type of issuer CLusterIssuer is to access the certificate across the namaspaces, Issuer is limited to within the namaspace.
    
    acme -> protocol used to obtain the certificates.
    
    server -> is where certificate request is sent to.
    
    email -> to receive the information of your certificate.
    
    privateKeySecretRef -> certificate information will be stored in a secrets.
    
    solvers -> cert-manager will listen to solver, in this case it is ingress

    http01 -> is called challenge,a challenge is necessary is to prove that you are the owner of the domain and server. with http01 you can get certificate only to that domain.
    (another example of challenge is DNS-01, which helps to get wild card certificate)

    Once the ClusterIssuer applied check for the status.
    ```
    kubectl get ClusterIssuer
    NAME                     READY   AGE
    letsencrypt-production   True    51s
    letsencrypt-staging      True    48s
    ```
6. Obtain the certificate
   Once ClusterIssuer status is True, Obtain the certificate
   replace <your_ip> with LB ip and apply the mentioned manifest
   ```
   kubectl apply -f certificate-staging.yaml
   kubectl apply -f certificate-production.yaml
    kubectl get certificate
    # NAME                  READY   SECRET                AGE
    # ssl-cert-production   True    ssl-cert-production   37s
    # ssl-cert-staging      True    ssl-cert-staging      47s
   ```
   Here, certificate will obtain tls certificate from the Issuer and stores in secrets.

7. Configure the Ingress to use the staging certificate
   Edit the ingress yaml with following details.
   ```
   tls:
    - hosts:
      - 51.8.64.189.nip.io
      secretName: ssl-cert-staging
   ```
   ```
   kubectl apply -f ingress.yaml
   ```
  If you access the certificate now, eventhough certificate is insecure, but we can see that dcertificate is issued to our domain.
  ![image](https://github.com/user-attachments/assets/df414fbe-0feb-4f52-aa45-2e0e4b6606cd)

 7. Configure the Ingress to user the Production certificate.
    Edit the ingress yaml with following details.
     ```
     tls:
      - hosts:
        - 51.8.64.189.nip.io
        secretName: ssl-cert-production
     ```
     ```
     kubectl apply -f ingress.yaml
     ```
     If we access the browser now, we can see the lock on top of browser.
    ![image](https://github.com/user-attachments/assets/b44103e6-3eca-4748-835d-2813a23c6c07)

    which means, we successfully installed certificate to our web application

    KUDUS.......

    References:
    1.  https://cert-manager.io/docs/installation/kubectl/
    2.  https://kubernetes.github.io/ingress-nginx/deploy/
    3.  https://kubernetes.github.io/ingress-nginx/deploy/#azure
    4.  https://learn.microsoft.com/en-us/cli/azure/aks/command?view=azure-cli-latest
    5.  

