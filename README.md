# node-todo-cicd

sudo apt install nodejs
sudo apt install npm


npm install

node app.js

Prometheus and Grafana Dashboard on EKS Cluster using Helm Chart.

PROJECT ARCHITECTURE:

<img width="2224" alt="grafana" src="https://user-images.githubusercontent.com/122585172/235338921-3cc88324-f038-4b69-a2b5-5cadb576aa85.png">

This blog explains how you can set up Prometheus and Grafana in Amazon EKS.

Kubernetes abstracts a lot of functionalities under the hood. Effective monitoring of such a dynamic system requires tools with advanced capabilities. Prometheus is one such application.

Prometheus is an open-source automated monitoring and alerting system. It has become a widely accepted tool for monitoring highly dynamic container environments such as Kubernetes and Docker Swarm. It can collect metrics from various sources, including containers, servers, and applications, and store them in a time-series database. Prometheus provides a flexible query language, called PromQL, that allows you to retrieve and analyze data. It also includes a web interface and an API for interacting with the data.

Grafana is a multi-platform that gets data from a data source such as Prometheus and transforms it into visualizations charts. We can create our own dashboards or use the existing ones provided by Grafana. We can personalize the dashboards as per our requirements.

Helm is the package manager for Kubernetes. Helm Charts help you define, install, and upgrade even the most complex Kubernetes application. Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste.

This Project will teach you how to integrate Prometheus and Grafana on Kubernetes using Helm


Setup an AWS EC2 Instance
Login to an AWS account using a user with admin privileges and ensure your region is set to us-east-1 N. Virginia.

Move to the EC2 console. Click Launch Instance.

For name use Main-Server

Select AMIs as Ubuntu and select Instance Type as t2.medium. Create new Key Pair and Create a new Security Group with traffic allowed from ssh, http and https.

![image](https://user-images.githubusercontent.com/122585172/235338965-38325ca4-1ecb-48c7-b53c-8fb301ca1f32.png)


Click on launch Instance and once EC2 Instance started, connect to it with EC2 Instance Connect.

Install AWS CLI and Configure
Now we need to set up the AWS CLI on the EC2 machine so that we can use eksctl in the later stages

Let us get the installation done for AWS CLI 2.

Linux x86(64-bit) If you are using Linux x86(64-bit) operating system:


curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
sudo apt install unzip
unzip awscliv2.zip 
sudo ./aws/install


![image](https://user-images.githubusercontent.com/122585172/235339009-923b5bab-4f5f-498b-bf93-39a8f939179a.png)


Okay now after installing the AWS CLI, let's configure the AWS CLI so that it can authenticate and communicate with the AWS environment.



Install and Setup Kubectl
Moving forward now we need to set up the kubectl also onto the EC2 instance.


curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version

![image](https://user-images.githubusercontent.com/122585172/235339025-33636908-78d3-44ae-a13f-01fbe8204bbf.png)



Install and Setup eksctl
Download and extract the latest release of eksctl with the following command.


curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp


Move the extracted binary to /usr/local/bin.

sudo mv /tmp/eksctl /usr/local/bin



Test that your installation was successful with the following command.

eksctl version


![image](https://user-images.githubusercontent.com/122585172/235339035-a8b2fe17-2056-4680-8455-58ad343038e4.png)


Install Helm chart
The next tool we need is Helm Chart. Helm is a package manager for Kubernetes, an open-source container orchestration platform. Helm helps you manage Kubernetes applications by making it easy to install, update, and delete them.

Install Helm Chart - Use the following script to install the helm chart -

$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh



Verify Helm Chart installation


![image](https://user-images.githubusercontent.com/122585172/235339058-37c261e6-115b-42c5-9b1e-55a073423eb2.png)


![image](https://user-images.githubusercontent.com/122585172/235339063-0cf92bc2-0dca-4761-9278-668d9a683c3e.png)



This way we install all AWS CLI, kubectl, eksctl and Helm.

![image](https://user-images.githubusercontent.com/122585172/235339092-c6308159-0c65-434c-8895-97f2190ae38a.png)


Creating an Amazon EKS cluster using eksctl
Now in this step, we are going to create Amazon EKS cluster using eksctl

You need the following in order to run the eksctl command

Name of the cluster : --eks4

Version of Kubernetes : --version 1.24

Region : --region us-east-1

Nodegroup name/worker nodes : --nodegroup-name worker-nodes

Node Type : --nodegroup-type t2.large

Number of nodes: --nodes 2

Minimum Number of nodes: --nodes-min 2

Maximum Number of nodes: --nodes-max 3

Here is the eksctl command -


eksctl create cluster --name eks2 --version 1.24 --region us-east-1 --nodegroup-name worker-nodes --node-type t2.large --nodes 2 --nodes-min 2 --nodes-max 3


![image](https://user-images.githubusercontent.com/122585172/235339110-7f96c77e-401c-42b8-bf46-f7165a7e11b6.png)


![image](https://user-images.githubusercontent.com/122585172/235339116-81bcbdc8-c1c1-4ef1-a7bf-ff0b0058b93a.png)


It took me 20 minutes to complete this EKS cluster. If you get any error for not having sufficient data for mentioned availability zone then try it again.


aws eks update-kubeconfig --name eks4



Verify the EKS Kubernetes cluster on AWS Console.

You can go back to your AWS dashboard and look for Elastic Kubernetes Service -> Clusters

![image](https://user-images.githubusercontent.com/122585172/235339125-192d4e12-e092-476d-a2a2-d1899193762f.png)



![image](https://user-images.githubusercontent.com/122585172/235339126-8edf8ca8-db7b-4ba8-bab4-1987a00015bf.png)



![image](https://user-images.githubusercontent.com/122585172/235339129-8f978cad-c04b-4e02-9cc1-966a28840d68.png)


![image](https://user-images.githubusercontent.com/122585172/235339134-a57dc3cf-bf3a-40f4-a1fc-659a98ef7706.png)



Installing the Kubernetes Metrics Server
Alright the next step would be to install the Kubernetes Metrics server onto the Kubernetes cluster so that Prometheus can collect the performance metrics of Kubernetes.

Deploy the Metrics Server with the following command:
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


![image](https://user-images.githubusercontent.com/122585172/235339144-39051ab2-764e-46c9-b22f-9d39d0de8b6f.png)



Verify that the metrics-server deployment is running the desired number of pods with the following command.


    kubectl get deployment metrics-server -n kube-system


![image](https://user-images.githubusercontent.com/122585172/235339155-eef0a5da-5bf4-4f7b-9b72-85e59a9b5d21.png)


Install Prometheus
Now install the Prometheus using the helm chart.

Add Prometheus helm chart repository

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts


Update the helm chart repository

helm repo update
helm repo list


Create prometheus namespace

kubectl create namespace prometheus


![image](https://user-images.githubusercontent.com/122585172/235339179-b55b81a8-67c1-4561-aa6a-07378aab45bb.png)


Install Prometheus


 helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"



![image](https://user-images.githubusercontent.com/122585172/235339187-bd82cbb6-1e4e-4970-a0e3-804d7ce281b2.png)



Create IAM OIDC Provider
Your cluster has an OpenID Connect (OIDC) issuer URL associated with it. To use AWS Identity and Access Management (IAM) roles for service accounts, an IAM OIDC provider must exist for your cluster's OIDC issuer URL.

When I run kubectl get all -n -prometheus.


![image](https://user-images.githubusercontent.com/122585172/235339192-e8f2f495-7283-4653-82c6-8837a5226bc6.png)



I noticed that not all servers are running. To fix this we are doing the following steps

oidc_id=$(aws eks describe-cluster --name eks4 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

eksctl utils associate-iam-oidc-provider --cluster eks4 --approve


Add IAM Role using eksctl with your cluster name.


eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster eks4 \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole



![image](https://user-images.githubusercontent.com/122585172/235339257-e6b663e8-3516-4da5-a939-cfd2c7ba0719.png)

Then add EBS CSI to eks by running the following command

Enter your account ID and cluster name.

eksctl create addon --name aws-ebs-csi-driver --cluster eks4 --service-account-role-arn arn:aws:iam::xxxxxxxxx:role/AmazonEKS_EBS_CSI_DriverRole --force


![image](https://user-images.githubusercontent.com/122585172/235339265-40f26a50-5562-4dcd-b8dd-861cd128e6c0.png)


Finally, all pods are running now.

![image](https://user-images.githubusercontent.com/122585172/235339277-c788f0aa-5299-4cf7-92f2-96e50ec0e710.png)


View the Prometheus dashboard by forwarding the deployment ports

![image](https://user-images.githubusercontent.com/122585172/235339288-b02dd7cd-00de-46dd-be59-30062bc4afa2.png)


Open different browser and connect to your EC2 instance and run curl localhost:9090/graph


![image](https://user-images.githubusercontent.com/122585172/235339297-bdfbecf1-4aa0-412b-a988-60076825bf92.png)


Install Grafana
Add the Grafana helm chart repository. Later, Update the helm chart repository.


helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update


Now we need to create a Prometheus data source so that Grafana can access the Kubernetes metrics. Create a yaml file prometheus-datasource.yaml and save the following data source configuration into it -


datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true


Create a namespace grafana

kubectl create namespace grafana


![image](https://user-images.githubusercontent.com/122585172/235339337-224310e3-9cf5-4c6b-97b1-dff6d7f3c12f.png)


Install the Grafana

helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values prometheus-datasource.yaml \
    --set service.type=LoadBalancer

This command will create the Grafana service with an external load balancer to get the public view.

![image](https://user-images.githubusercontent.com/122585172/235339362-f95fce9a-73bc-4bb7-9c50-cf91dc2cd004.png)


Verify the Grafana installation by using the following kubectl command -

![image](https://user-images.githubusercontent.com/122585172/235339367-cb431f83-11ea-4999-a84c-dcd98c47aea7.png)



Copy External IP address and open it in the browser -

Password you mentioned as EKS!sAWSome while creating Grafana

![image](https://user-images.githubusercontent.com/122585172/235339373-8e8c020b-49c9-45f6-abe6-cfdf0ef1a9de.png)



Import Grafana dashboard from Grafana Labs
Now we have set up everything in terms of Prometheus and Grafana. For the custom Grafana Dashboard, we are going to use the open source grafana dashboard. For this session, I am going to import a dashboard 6417


![image](https://user-images.githubusercontent.com/122585172/235339386-f2e7e3cc-1e5c-463b-9d85-ffabcf555bca.png)


Load and select the source as Prometheus



![image](https://user-images.githubusercontent.com/122585172/235339390-1b9a2190-a39c-4895-baea-f8b5cd360c94.png)


Import it.

![image](https://user-images.githubusercontent.com/122585172/235339396-23ab6996-6416-42ab-9e8c-f436f6bad34c.png)



Deploy a Node.js application and monitor it on Grafana
To make use of Grafana dashboard, we will deploy Node.js application on Kubernetes. Download deployment.yml file from the my repository.



To deploy the Node.js application on kubernetes cluster user the following kubectl command. Verify the deployment by running the following kubectl command

kubectl apply -f deployment.yml
kubectl get deployment
kubectl get pods

![image](https://user-images.githubusercontent.com/122585172/235339428-3016340d-2fe5-4455-8e5c-2f6bf0d94cf3.png)


![image](https://user-images.githubusercontent.com/122585172/235339430-8a1ae7ea-9355-4b2f-bd59-be81488dd212.png)


The Node.js Application is running successfully.

![image](https://user-images.githubusercontent.com/122585172/235339437-17d007a7-c6cf-498b-9a6a-dc71382ede1d.png)



Refresh the Grafana dashboard to verify the deployment

![image](https://user-images.githubusercontent.com/122585172/235339442-794ff698-5f7d-40cb-adf1-7b524ecb5528.png)



![image](https://user-images.githubusercontent.com/122585172/235339447-aa4f6f04-cb92-4b19-aedc-8281c090f859.png)


![image](https://user-images.githubusercontent.com/122585172/235339451-e7996987-59a1-4897-9367-82b001dad4e2.png)


Clean Up
In this stage, you're going to clean up and remove all resources which we created during the session. So that it will not be charged to you afterward.

Delete EKS cluster with following command.

eksctl delete cluster --name eks4


![image](https://user-images.githubusercontent.com/122585172/235339467-078f480c-3474-4d83-835e-e0a0ef307222.png)


Delete EC2 Instance.



