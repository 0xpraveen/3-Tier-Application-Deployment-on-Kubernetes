# 3-Tier-Application-Deployment-on-Kubernetes

<img width="919" alt="three-tier diagram" src="https://github.com/0xpraveen/3-Tier-Application-Deployment-on-Kubernetes/assets/130580134/10440af0-03fe-4fee-971b-df80895cefde"> </img>

## Prerequisites
- Basic knowledge of Docker, and AWS services.
- An AWS account with necessary permissions.

## Application Code
The `Application-Code` directory contains the source code for the Three-Tier Web Application. Dive into this directory to explore the frontend and backend implementations.

## Kubernetes Manifests Files
The `Kubernetes-Manifests-Files` directory holds Kubernetes manifests for deploying your application on AWS EKS. Understand and customize these files to suit your project needs.

## Project Details
- IAM User setup.
- EKS Cluster creation & Load Balancer configuration.
- Private ECR repositories for secure image management.
- Helm charts for efficient monitoring setup.

## Getting Started

### Step 1: IAM Configuration.
- Create a user eks-admin with AdministratorAccess.
- Generate Security Credentials: Access Key and Secret Access Key.

### Step 2: EC2 Setup
- Launch an Ubuntu instance in your favourite region (eg. region us-west-2).
- SSH into the instance from your local machine.

### Step 3: Install AWS CLI v2
```
curl https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o awscliv2.zip
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```
### Step 4: Install Docker
```
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```
### Step 5: Install kubectl
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
### Step 6: Install eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

```
### Step 7: Setup EKS Cluster
```
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```

### Step 8: Run Manifests
```
kubectl create namespace workshop
kubectl apply -f .
kubectl delete -f .
```
### Step 9: Install AWS Load Balancer
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::666072234343:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```
### Step 10: Deploy AWS Load Balancer Controller
```
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f ingress.yaml
```
<img width="1440" alt="Screenshot 2024-03-16 at 10 40 59 PM" src="https://github.com/0xpraveen/3-Tier-Application-Deployment-on-Kubernetes/assets/130580134/ba6902ba-d315-416f-98c1-9b1254fb61b1"> </img>


### Cleanup
- To delete the EKS cluster:
  ```
  eksctl delete cluster --name three-tier-cluster --region us-west-2
