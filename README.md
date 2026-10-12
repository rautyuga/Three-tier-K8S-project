# #TWSThreeTierAppChallenge

## Overview

This repository hosts the **#TWSThreeTierAppChallenge** The challenge involves deploying a Three-Tier Web Application using **ReactJS**, **NodeJS**, and **MongoDB**, with deployment on **AWS EKS**.

## Prerequisites

- Basic knowledge of Docker and AWS services.
- An AWS account with the necessary permissions.

## Challenge Steps

- Application Code
- Kubernetes Manifests Files

## Project Details

### Application Code

The `Application-Code` directory contains the source code for the Three-Tier Web Application. Explore this directory for the frontend and backend implementations.

### Kubernetes Manifests Files

The `Kubernetes-Manifests-Files` directory holds the Kubernetes manifests for deploying the application on AWS EKS. Understand and customize these files to suit your project needs.

### Tools Explored

- AWS CLI for AWS access and configuration
- Docker, kubectl and eksctl for building, deploying and managing the cluster
- Helm

### High-Level Overview

- IAM user setup on AWS
- EKS cluster creation and Load Balancer configuration
- Private ECR repositories for secure image management
- Helm charts

The journey covers setting up the tools, deploying a Three-Tier app and ensuring data persistence.

## Getting Started

Follow the steps below to set up IAM, create the EKS cluster and deploy the application.

### Step 1: IAM Configuration

1. Create a user `eks-admin` with `AdministratorAccess`.
2. Generate security credentials: Access Key and Secret Access Key.

### Step 2: EC2 Setup

1. Launch an Ubuntu instance in your favourite region (e.g. `us-west-2`).
2. SSH into the instance from your local machine.

### Step 3: Install AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### Step 4: Install Docker

```bash
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### Step 5: Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 6: Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 7: Set Up the EKS Cluster

```bash
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```

### Step 8: Run the Manifests

```bash
kubectl create namespace workshop
kubectl apply -f .
```

To remove the resources again:

```bash
kubectl delete -f .
```

### Step 9: Install the AWS Load Balancer Controller Prerequisites

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```

Replace `<AWS_ACCOUNT_ID>` with your own 12-digit AWS account ID.

### Step 10: Deploy the AWS Load Balancer Controller

```bash
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=three-tier-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f full_stack_lb.yaml
```

## Cleanup

Delete the EKS cluster:

```bash
eksctl delete cluster --name three-tier-cluster --region us-west-2
```

To clean up the rest and avoid incurring any cost:

1. Stop or terminate the EC2 instance created in Step 2.
2. Delete the Load Balancer created in Steps 9 and 10.
3. Open the EC2 console, go to the Security Groups section and delete the security groups created in the previous steps.

# Three-tier-K8S-project
DevOps-Projec
