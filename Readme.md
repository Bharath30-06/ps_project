# Helm Chart Deployment Guide

This guide outlines the steps to deploy an application using **Helm** on an EC2 instance.

---

## Step 1: Dockerize the Application

1. Create `Dockerfile` and `docker-compose.yml` for your application.
2. Build and run containers locally:
   
   docker compose up -d
Step 2: Push Docker Images to Docker Hub
Tag the image:
docker tag <local-image>:<tag> <dockerhub-username>/<repo-name>:<tag>
Push the image:
docker push <dockerhub-username>/<repo-name>:<tag>

Step 3: Create Helm Chart
Create deployment files in the templates folder of your Helm chart.

Add values in the values.yaml file.

Step 4: Prepare EC2 Instance
Launch a Linux EC2 instance, then SSH into it:


ssh -i <your-key>.pem ec2-user@<ec2-public-ip>
Install Required Tools
Install Git:

sudo yum install git -y
Use the following script to install kubectl, aws-cli, and eksctl:

#!/bin/bash

set -e

# Update and install dependencies
sudo apt-get update -y
sudo apt-get install -y curl unzip

# Install kubectl
KUBECTL_VERSION="1.30.4"
KUBECTL_DATE="2024-09-11"
curl -O "https://s3.us-west-2.amazonaws.com/amazon-eks/${KUBECTL_VERSION}/${KUBECTL_DATE}/bin/linux/amd64/kubectl"
curl -O "https://s3.us-west-2.amazonaws.com/amazon-eks/${KUBECTL_VERSION}/${KUBECTL_DATE}/bin/linux/amd64/kubectl.sha256"
sha256sum -c kubectl.sha256
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
kubectl version --client

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm -rf aws awscliv2.zip

# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
Step 5: Create EKS Cluster
Use the following command to create your EKS cluster:
eksctl create cluster \
  --name test-cluster \
  --version 1.30 \
  --region ap-south-1 \
  --nodegroup-name test-linux \
  --node-type t2.medium \
  --nodes 2
Step 6: Install Helm

sudo snap install helm --classic
Step 7: Clone Your Repository

git clone <your-git-repo-url>
cd <repo-directory>
Step 8: Deploy Helm Chart

helm create helm_files
helm install helm_deploy helm_files
