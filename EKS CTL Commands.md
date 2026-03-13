## Pre-Requisites
```
IAM USER with below permissions and along with that access and security key.
AmazonEKSClusterPolicy
AmazonEKSWorkerNodePolicy
AmazonEC2FullAccess
AmazonVPCFullAccess
AWSCloudFormationFullAccess
IAMFullAccess

aws configure
aws sts get-caller-identity
```

## Install kubectl command

```
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl

# Make it executable:
chmod +x kubectl

# Move it to system path:
sudo mv kubectl /usr/local/bin/

# Verify installation:
kubectl version --client
```

## Install ekstcl command
```
# Run this command:
curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz"

# Extract it:
tar -xzf eksctl_$(uname -s)_amd64.tar.gz

# Move it to /usr/local/bin:
sudo mv eksctl /usr/local/bin

# Verify installation:
eksctl version
```

## Step 01: Create EKS Cluster using eksctl

```bash
# Create Cluster
eksctl create cluster --name=juhisinha \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup                  
```

```bash
# Get List of clusters
eksctl get cluster

# Configure kubectl for your EKS cluster
aws eks update-kubeconfig --region us-east-1 --name juhisinha
```

## Step 02: Create & Associate IAM OIDC Provider for our EKS Cluster
* To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create & associate OIDC identity provider.
* To do so using eksctl we can use the below command.

```bash
# Template
eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluter-name> \
    --approve

# Replace with region & cluster name
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster juhisinha \
    --approve
```

## Step 03: Create EC2 Keypair
* Create a new EC2 Keypair with name as amc-demo
* This keypair we will use it when creating the EKS NodeGroup.
* This will help us to login to the EKS Worker Nodes using Terminal.

## Step 04: Create Node Group with additional Add-Ons in Public Subnets
These add-ons will create the respective IAM policies for us automatically within our Node Group role.

```bash
# Create Public Node Group   
eksctl create nodegroup --cluster=juhisinha \
                       --region=us-east-1 \
                       --name=juhi-ng-public1 \
                       --node-type=t3.medium \
                       --nodes=2 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=AWS_KeyPair \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```

## Step 05: Verify Cluster & Nodes

```bash
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=<clusterName>

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# Our kubectl context should be automatically changed to new cluster
kubectl config view --minify
```

## Step 06: Delete Node Group

```bash
# List EKS Clusters
eksctl get clusters

# Capture Node Group name
eksctl get nodegroup --cluster=<clusterName>
eksctl get nodegroup --cluster=amcdemo

# Delete Node Group
eksctl delete nodegroup --cluster=<clusterName> --name=<nodegroupName>
eksctl delete nodegroup --cluster=amcdemo --name=amcdemo-ng-public
```

## Step 07: Delete Cluster
```bash
# Delete Cluster
eksctl delete cluster <clusterName>
eksctl delete cluster amcdemo
```
