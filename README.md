# OpenFaasEKS
Eks Cluster with OpenFaas deployment
-In this repo, you will find terraform code for EKS Cluster creation and below, you will find steps to deploy an OpenFaas onto the cluster using Helm.


SOLUTION.

#EKS CLUSTER CREATION USING TERRAFORM
1) Clone Git repo 
2) Run "terraform init"
3) Run "terraform plan" to review resources to be created 
4) Run "terraform apply"

#INSTALL HELM and Deploy OpenFaas 

1) Run "brew install helm"
2) Create Kubernetes Namespaces for OpenFaaS using the following command:

$ kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml

3) Prepare Authentication for OpenFaaS by generating a random password by reading random bytes, then create a hash:

 $ PASSWORD=$(head -c 12 /dev/urandom | shasum | cut -d' ' -f1)

4) Create/customize username by running the command below:

$ kubectl -n openfaas create secret generic basic-auth \
  --from-literal=basic-auth-user=admin \
  --from-literal=basic-auth-password=$PASSWORD

5) Add the OpenFaaS Helm chart repo: 

  $ helm repo add openfaas https://openfaas.github.io/faas-netes/

6) Install OpenFaaS with authentication enabled by default using Helm:

 $ helm upgrade openfaas --install openfaas/openfaas \
    --namespace openfaas  \
    --set functionNamespace=openfaas-fn \
    --set serviceType=LoadBalancer \
    --set basic_auth=true \
    --set operator.create=true \
    --set gateway.replicas=2 \
    --set queueWorker.replicas=2

7) Run the following command to verify deployment :

 $ kubectl --namespace=openfaas get deployments -l "release=openfaas, app=openfaas"


8) A LoadBalancer will be created for your OpenFaaS Gateway. It may take 10-15 minutes for this to be provisioned, then the public IP address can be found with this command:

  $ kubectl get svc -n openfaas -o wide

9) When the EXTERNAL-IP field shows an address, you can save the value into an environmental variable for use with the CLI 

  $ export OPENFAAS_URL=$(kubectl get svc -n openfaas gateway-external -o  jsonpath='{.status.loadBalancer.ingress[*].hostname}'):8080  \
    && echo Your gateway URL is: $OPENFAAS_URL

10) Open Public address and log into OpenFaas.


