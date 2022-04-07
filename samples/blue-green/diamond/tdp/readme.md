## Instructions

### Blue Deployment
1. Deploy blue namespace 

    Define environment variables for blue
    `export TARGET_ROLE=blue`
    `export IMAGE_VERSION=blue`

    Run a script to apply the environment variables to the deployment files
    `kubectl apply -f nginx-blue.namespace.yaml`  
    `cat nginx.deployment.yml | sh config.sh | kubectl create --save-config -f -`
    `kubectl create -f nginx-blue.service.yml`

### Green Deployment
2. Deploy green namespace

    Define environment variables for green
    `export TARGET_ROLE=green`
    `export IMAGE_VERSION=green`

    Run a script to apply the environment variables to the deployment files
    `kubectl apply -f nginx-green.namespace.yaml`
    `cat nginx.deployment.yml | sh config.sh | kubectl create --save-config -f -`   
    `kubectl create -f nginx-green.service.yml`

### Production Deployment
3. Deploy production namespace
    `kubectl apply -f production.namespace.yaml`
    `kubectl apply -f production.nginx.service.blue.yaml`
    `kubectl apply -f production.nginx.service.green.yaml`

### Blue-Green Deployment
4. Set production to point to blue
    `kubectl apply -f production.ingress.blue.yaml`

5. Set production to point to green
    `kubectl apply -f production.ingress.green.yaml`




