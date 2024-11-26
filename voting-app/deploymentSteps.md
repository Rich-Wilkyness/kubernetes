# pt 1 without Deployments
1. Create all the necessary files. For this demo, the pods and services
2. Start miniKube 
3. Create the pods and services, and then check the status of the pods and services (ensure they are running)
4. If you want to get the url of the service, you can use the following command: 
 - minikube service <service-name> --url
 - you will need 1 url for the voting app and 1 url for the result app
5. Go to the url and you should see the voting app running

# pt 2 w/ Deployments
1. Create all the necessary files. For this demo, the deployments and services
2. Start miniKube
3. Create the deployments and services, and then check the status of the deployments and services (ensure they are running)
4. If you want to get the url of the service, you can use the following command: 
 - minikube service <service-name> --url
 - you will need 1 url for the voting app and 1 url for the result app
 - if running multiple pods on a deployment, use a LoadBalancer type in the service yaml file instead of NodePort 



- On hosted Kubernetes clusters (GKE, EKS, AKS) you cannot access the master node
# pt 3 GKE Deployment
1. Create all the necessary files. For this demo, the deployments and services
2. Make Google Cloud account and create a project (https://cloud.google.com/free/docs/gcp-free-tier) - 12 month free trial
3. Create a cluster in Google Cloud and keep the default settings for now
4. Open the Google Cloud Shell and run: 
    - gcloud container clusters get-credentials <cluster-name> --zone <zone> --project <project-name>
    - kubectl get nodes (to check if the cluster is running)
5. Clone the repo and cd into the directory with the yaml files
6. Create the deployments and services, and then check the status of the deployments and services (ensure they are running)
    - if you want to create all of the files at once, you can use the following command: 
        - "kubectl create -f <directory-name>" or the current directory "kubectl create -f ."

# [pt 4 EKS Deployment](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)
- Pre-requisites:
    1. AWS account (https://aws.amazon.com/free/)
    2. Install KubeCtl CLI
    3. AWS Basics
        - EKS Cluster Role
        - IAM Role for Node Group
        - VPC
        - E2C Key Pair which can be used to SSH to the worker nodes
1. Go to the AWS Management Console search for EKS, this will take you to the EKS Dashboard to create a cluster
2. Create a cluster and keep the default settings and provide the pre-requisite AWS resources (EKS Cluster Role)
    - can take up to 15 minutes to create the cluster, so be patient
3. Go to the EKS Dashboard and click on the cluster you just created, and then click on the "Compute" tab to create a node group
    - provide the pre-requisite AWS resources (IAM Role for Node Group and Key Pair (only need this if you want to SSH into the worker nodes))
    - keep the default settings for everything else
4. Go to terminal and setup kubectl utility to access the cluster
    - AWS CLI has instructions on how to interact the our local minikube cluster
    - also need to install AWS CLI if you haven't already
5. Get access to the cluster by running the following command: 
    - aws eks --region <region> update-kubeconfig --name <cluster-name>
    - kubectl get nodes (to check if the cluster is running)
6. Clone the repo and cd into the directory with the yaml files
7. Create the deployments and services, and then check the status of the deployments and services (ensure they are running)
    - if you want to create all of the files at once, you can use the following command: 
        - "kubectl create -f <directory-name>" or the current directory "kubectl create -f ."
8. Check the status of the pods and services to ensure they are running
    - kubectl get deployments,svc
9. Use the LoadBalancer url to access the voting app and result app
    - kubectl get svc, copy the url and paste it into your browser
10. If you want to delete the deployments and services, you can use the following command: 
    - "kubectl delete -f <directory-name>" or the current directory "kubectl delete -f ."
    - you'll want to do this to avoid charges on your AWS account

# [pt 5 AKS Deployment](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal)
- Pre-requisites: 
    1. Azure account (https://azure.microsoft.com/en-us/free/free-account-faq/) - 12 month free trial
    2. Active Azure subscription
    3. Azure Basics
        - Resource Group
        - AKS Cluster
        - ACR (Azure Container Registry)
1. Go to the Azure Portal and search for AKS (Kubernetes services), this will take you to the AKS Dashboard to create a cluster
2. Click on "Add" to create a new cluster 
    - name a resource group, cluster name, region (default), and node size (1)
    - Authentication: Service Principal (default), this will create a new service principal for you, and manages cloud resources on your behalf for the cluster
    - Review and Create (this will take a few minutes to create the cluster)
3. Search in the Search Bar for the name of the cluster you just created and click on it (it will be under Resources)
4. Open the cloud shell (to the right of the search bar) 
    - you might need to create a storage account, if you haven't already done so (this is free) 
    - kubectl is already installed in the cloud shell
5. Connect to the cluster by running the following command: 
    - az aks get-credentials --resource-group <resource-group> --name <cluster-name>
    - kubectl get nodes (to check if the cluster is running)
6. Clone the repo and cd into the directory with the yaml files
7. Create the deployments and services, and then check the status of the deployments and services (ensure they are running)
    - if you want to create all of the files at once, you can use the following command: 
        - "kubectl create -f <directory-name>" or the current directory "kubectl create -f ."
8. Check the status of the deployments and services to ensure they are running
    - kubectl get deployments,svc
9. Use the LoadBalancer url to access the voting app and result app
    - kubectl get svc, copy the url and paste it into your browser
10. Delete the deployments and services to avoid charges/credit on your Azure account
    - "kubectl delete -f <directory-name>" or the current directory "kubectl delete -f ."