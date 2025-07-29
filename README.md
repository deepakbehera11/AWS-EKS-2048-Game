# AWS-EKS-2048-Game

This repository provides step-by-step instructions to deploy the classic 2048 game on AWS Elastic Kubernetes Service (EKS) using Fargate and the AWS Load Balancer Controller.

Checkout my blog for step by step guide -> [Deploying the Classic 2048 Game on AWS EKS](https://deepakbehera.hashnode.dev/deploying-the-classic-2048-game-on-aws-elastic-kubernetes-service-eks).

## Prerequisites

- AWS CLI installed and configured
- `kubectl` installed
- `eksctl` installed
- Helm installed (for the AWS Load Balancer Controller)
- AWS Account IAM permissions

See [installation.md](installation.md) for details.

## Setting Up Your AWS Environment
### Configuring AWS CLI
```
aws configure
```
You'll be prompted to enter:

AWS Access Key ID ~ Your Access Key

AWS Secret Access Key ~ Your Secret Access Key

Default region (e.g., us-east-1)

## 1. Create the EKS Cluster Using Fargate

```sh
eksctl create cluster --name cluster-2048-game --region us-east-1 --fargate

```
## 2.  Configure kubectl for EKS
```
aws eks update-kubeconfig --name cluster-2048-game --region us-east-1
```
## 3. Creating a Fargate Profile
```
eksctl create fargateprofile \
    --cluster cluster-2048-game \
    --region us-west-1 \
    --name alb-sample-app \
    --namespace game-2048
```
## 4. Deploying the 2048 Game
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
## 5. Configuring IAM OIDC Provider
```
eksctl utils associate-iam-oidc-provider --cluster cluster_name --approve --region region_name
```
## 6. Creating IAM Policy for the Load Balancer Controller
```
curl -O <https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json>
```
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
## 7. Creating IAM Service Account
```
eksctl create iamserviceaccount \\
  --cluster=<your-cluster-name> \\
  --namespace=kube-system \\
  --name=aws-load-balancer-controller \\
  --role-name AmazonEKSLoadBalancerControllerRole \\
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \\
  --approve

Remember to replace  with your actual AWS account ID.
```
## 8. Installing the Load Balancer Controller with Helm
```
# Add the EKS chart repository
helm repo add eks <https://aws.github.io/eks-charts>

# Update the repository
helm repo update eks
```
``` 
# Install the controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \\
  --set clusterName=<your-cluste-name> \\
  --set serviceAccount.create=false \\
  --set serviceAccount.name=aws-load-balancer-controller \\
  --set region=us-east-1 \\
  --set vpcId=<your-vpc-id>
```
## 9. Verifying the Controller Deployment
```
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl get pods -n kube-system -w
```
## 10. Accessing the 2048 Game
```
kubectl get ingress -n game-2048
```
The output should include an ADDRESS field with a DNS name. Open this address in your web browser, and you should see the 2048 game running!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752345512993/a666530b-06d9-44bb-b30f-c04cce3ef078.png?auto=compress,format&format=webp)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752345626756/ba866c85-c9b2-49c5-8248-d45fc85c8b0e.png?auto=compress,format&format=webp)

## 11. Deletion
```
eksctl delete cluster --name demo-cluster --region us-east-1
```
It can take time sometimes to delete everything, we need to be patient here

For billing Purposes to not get charged, you can check the following and delete anything related to the creation of the services :
- VPC
- Cloud Watch - logs
- EKS
- ECR
- security groups
- IAM Access Keys