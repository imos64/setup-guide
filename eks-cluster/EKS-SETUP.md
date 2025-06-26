# Amazon EKS Cluster Setup Guide

This guide provides a comprehensive walkthrough for creating and managing an Amazon Elastic Kubernetes Service (EKS) cluster from your local machine using `eksctl`.

---

## 🚀 Prerequisites

Ensure the following tools are installed on your local machine:

- Linux system with `curl`, `unzip`, and `sudo` installed
- x86_64 architecture

## Installation Guide For AWS CLI, kubectl, and eksctl (Linux x86_64)
```bash
1. Install AWS cli
##Download the AWS CLI v2 ZIP installer
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

##Unzip the downloaded package
unzip awscliv2.zip

##Run the installer
sudo ./aws/install

##Verify the installation
aws --version


2. Install kubectl 
##Download the latest stable release of kubectl
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

##Make the binary executable
chmod +x kubectl

##Move the binary to your system's PATH
sudo mv kubectl /usr/local/bin/

##Verify the installation
kubectl version --client


3. Install eksctl 
##Download and extract the latest release to /tmp
curl --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

##Move the binary to your system's PATH
sudo mv /tmp/eksctl /usr/local/bin

##Verify the installation
eksctl version


### ✅ Verify Installation

```bash
aws --version
kubectl version --client
eksctl version
```
Configure AWS credentials using:
---

## 🔐 Configure AWS CLI

Run the following command and provide your credentials:

```bash
aws configure
```

Input:

- **AWS Access Key ID**
- **AWS Secret Access Key**
- **Default region name** (e.g., `us-east-1`)
- **Output format** (e.g., `json`)

Verify your identity:

```bash
aws sts get-caller-identity
```

---

## 📆 IAM Permissions Required

Ensure your IAM user/role has the following permissions:

- `eks:*`
- `ec2:*`
- `iam:*`
- `cloudformation:*`
- `autoscaling:*`

Attach the following AWS managed policies:

- `AmazonEKSClusterPolicy`
- `AmazonEKSWorkerNodePolicy`
- `AmazonEKSVPCControllerPolicy`
- `AmazonEC2ContainerRegistryReadOnly`
- `AmazonEKSVPCControllerPolicy`
- `EKSAdminFullAccess`

---

## 📆 Create an EKS Cluster

Use `eksctl` to create the EKS cluster:

```bash
eksctl create cluster \
  --name my-eks-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

> ⏱ This process takes approximately 10–15 minutes.

This command will:

- Provision a VPC and subnets
- Create the EKS control plane
- Launch a managed node group
- Set up IAM roles and policies

---

## 🔄 Configure `kubectl`

Update your kubeconfig to connect `kubectl` with your new cluster:

```bash
aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster
```

Verify the cluster:

```bash
kubectl get nodes
```

You should see the worker nodes listed and in a `Ready` state.

---

## 🌐 Deploy a Sample Application

Test your cluster by deploying a simple NGINX server:

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=LoadBalancer
kubectl get svc
```

Once the `EXTERNAL-IP` is available, open it in your browser to verify.

---

## 🛠️ Optional: Enable IAM OIDC Provider

Enable IAM OIDC provider for IRSA (IAM Roles for Service Accounts):

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster my-eks-cluster \
  --approve
```

---

## 🚮 Clean Up Resources

To delete the cluster and all associated AWS resources:

```bash
eksctl delete cluster --name my-eks-cluster --region us-east-1
```

---

## 📂 Troubleshooting

| Issue                             | Solution                                            |
| --------------------------------- | --------------------------------------------------- |
| `kubectl get nodes` returns empty | Re-run `aws eks update-kubeconfig`                  |
| Access denied errors              | Check IAM policies and permissions                  |
| LoadBalancer has no `EXTERNAL-IP` | Wait 1–2 minutes or check ELB status in AWS Console |

---

## 📒 References

- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [eksctl GitHub Repository](https://github.com/eksctl-io/eksctl)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)

---

## 💡 Tips

- Use `--profile <profile-name>` for multiple AWS profiles.
- Automate production provisioning with Terraform or AWS CDK.
- Consider integrating GitOps tools like ArgoCD or FluxCD.

