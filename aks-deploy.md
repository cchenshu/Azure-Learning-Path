## Install docker environment
## Build docker image and test locally
- `cd file-upload-api`
- `docker compose up -d`
- Open a browser and type http://0.0.0.0:5000
- To check the container log:
  - Run `docker ps` to get the container id
  - Run `docker logs -f <container id>`

## Push the docker image to ACR (private container images)
- Change the tag of the image
  - `docker tag xx.microsoft.com/testapp:v1 myregistry.azurecr.io/testapp:v1`
- Log in your azure account if needed
  - `az login`
  - `az account set --subscription xxxx-xxxx-xxxx....`
- Log in to ACR
  - `az acr login --name myregistry`
- Push the image and check the image in the ACR in the Azure portal
  - `docker push myregistry.azurecr.io/testapp:v1`

## Deploy to AKS and test the APIs
- delete pods(if cannot delete the pod follow this [link](https://www.cnblogs.com/landminejue/p/15459532.html))
  - `kubectl get pods -A`
  - `kubectl get deployment -n default (default is the namespace)`
  - `kubectl delete deployment azure-form-recognizer-read -n default`
     
- `kubectl get nodes` information about pools if you create a new pool(testai) and also get name for new pool
- `kubectl label nodes aks-testai-30110567-vmss000003 node=testai` label the node
- `kubectl apply -f azure-app-deployment.yaml`    
- `kubectl apply -f azure-app-service.yaml`
- `kubectl get service azure-app`
  
- Log in to Azure portal and and click: 
  - aks-cluster -> Services and ingresses -> fileupload -> External IP  
  - Run your test in the opened brower tab

