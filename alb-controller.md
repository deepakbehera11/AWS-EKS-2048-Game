# Creating IAM Policy for the Load Balancer Controller
Download IAM policy
```
curl -O <https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json>
```
Create IAM Policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
Create IAM Role
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
# Deploy ALB controller
Add helm repo
```
helm repo add eks <https://aws.github.io/eks-charts>
```
Update the repo
```
helm repo update eks
```
Install

```
# Install the controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \\
  --set clusterName=<your-cluste-name> \\
  --set serviceAccount.create=false \\
  --set serviceAccount.name=aws-load-balancer-controller \\
  --set region=us-east-1 \\
  --set vpcId=<your-vpc-id>

```
Verify that the deployments are running.
```
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl get pods -n kube-system -w
```