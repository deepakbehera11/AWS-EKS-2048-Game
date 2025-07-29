## Creating the EKS Cluster Using Fargate
```
eksctl create cluster --name cluster-2048-game --region us-east-1 --fargate
```
## Configuring kubectl for EKS
```
aws eks update-kubeconfig --name cluster-2048-game --region us-east-1
```
## Creating a Fargate Profile
```
eksctl create fargateprofile \
    --cluster cluster-2048-game \
    --region us-west-1 \
    --name alb-sample-app \
    --namespace game-2048
```
## Deploying the 2048 Game
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
verify that our resources were created successfully
```
kubectl get pods -n game-2048
kubectl get svc -n game-2048
kubectl get ingress -n game-2048
```
