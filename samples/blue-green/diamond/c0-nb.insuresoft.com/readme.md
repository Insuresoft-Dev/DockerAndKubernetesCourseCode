## Instructions

### Prerequisites
   1. Setup resource group with Azure Container Registry and Azure Kubernetes Services
    
        ```
        az login --service-principal --username ?? --password ?? --tenant ??
        az group create --name myResourceGroup --location southcentralus
        az acr create --resource-group myResourceGroup --name mytenataksacr  --sku Basic
        az role assignment create --assignee ?? --role contributor --resource-group myResourceGroup
        az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 2 --enable-addons monitoring --generate-ssh-keys --attach-acr mytenataksacr
        ```

   2. Allow kubectl commands in Azure Cloud Shell
    
        ```
        az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
        ```

   3. Build and Push application images to Azure in Visual Studio Code
    
        ```
        az acr login --name mytenataksacr.azurecr.io
        ```
        
        Right click 537.004.100-0/latest image and select push and and select the Azure - Lab/mytenantaksacr and  hit enter  

        Right click 537.006.100-0/latest image and select push and and select the Azure - Lab/mytenantaksacr and  hit enter

   4. Install kubernetes default ingress-nginx controller for using the ingress below
    
        https://kubernetes.github.io/ingress-nginx/deploy/
        
        https://github.com/kubernetes/ingress-nginx
        
        ```
        kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/cloud/deploy.yaml
        ```

   5. Before pulling the docker image from the Azure Container Registry must register a secret
    
        ```
        kubectl create secret docker-registry mytenataksacr-registry-connection --docker-server=mytenataksacr.azurecr.io --docker-username=?? --docker-password=??
        ```

### 537.004.100-0 Deployment
1. Deploy 537-004-100-0 namespace 

    Define environment variables for 537-004-100-0
    
    ```
    export TARGET_ROLE=537-004-100-0
    export IMAGE_VERSION=537.004.100-0
    ```

    Run a script to apply the environment variables to the deployment files
    
    ```
    kubectl apply -f 537-004-100-0.namespace.yaml
    cat nginx.deployment.yaml | sh config.sh | kubectl create --save-config -f -
    kubectl create -f 537-004-100-0.service.yaml
    ```

### 537.006.100-0 Deployment
2. Deploy 537.006.100-0 namespace

    Define environment variables for 537-006-100-0
    
    ```
    export TARGET_ROLE=537-006-100-0
    export IMAGE_VERSION=537.006.100-0
    ```

    Run a script to apply the environment variables to the deployment files
    
    ```
    kubectl apply -f 537-006-100-0.namespace.yaml
    cat nginx.deployment.yaml | sh config.sh | kubectl create --save-config -f -
    kubectl create -f 537-006-100-0.service.yaml
    ```

### Production Deployment
3. Deploy production namespace
    
    ```
    kubectl apply -f production.namespace.yaml
    kubectl apply -f production.nginx.537-004-100-0.service.yaml
    kubectl apply -f production.nginx.537-006-100-0.service.yaml
    ```

    Get the ingress nginx controller external ip
    
    ```
    kubectl get svc ingress-nginx-controller -n ingress-nginx
    ```
    
    Add the following line to C:\Windows\System32\drivers\etc\hosts
    
    EXTERNAL-IP diamond.insuresoft.com
    
    EXTERNAL-IP 537-004-100-0.insuresoft.com
    
    EXTERNAL-IP 537-006-100-0.insuresoft.com

### Blue-Green Deployment

#### Blue
4. Set production to point to 537-004-100-0
    
    ```
    kubectl apply -f production.537-004-100-0.ingress.yaml
    ```
    
    http://diamond.insuresoft.com/
    
    http://537-004-100-0.insuresoft.com/
    
    http://537-006-100-0.insuresoft.com/
    
    http://diamond.insuresoft.com/537-004-100-0
    
    http://diamond.insuresoft.com/537-006-100-0

#### Green
5. Set production to point to 537-006-100-0
    
    ```
    kubectl apply -f production.537-006-100-0.ingress.yaml
    ```
    
    http://diamond.insuresoft.com/
    
    http://537-004-100-0.insuresoft.com/
    
    http://537-006-100-0.insuresoft.com/
    
    http://diamond.insuresoft.com/537-004-100-0
    
    http://diamond.insuresoft.com/537-006-100-0




