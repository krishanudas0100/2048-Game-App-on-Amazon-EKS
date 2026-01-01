# 2048-Game-App-on-Amazon-EKS
2048 Game App on Amazon EKS

Prerequisites:
Need to install 
1.Kubectl:
2.Kubectl:
3.AWS CLI
4.eksctl

# Set architecture (use amd64 for most systems, or arm64 for ARM)
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
# Download the latest release
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
# Extract and install
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl


create eks cluster:
eksctl create cluster --name demo-cluster-1 region ap-south-1 --fargate

need to add fargate profile:

eksctl create fargateprofile \
    --cluster demo-cluster-2 \
    --region us-east-2 \
    --name alb-sample-app \
    --namespace game-2048
	
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml


Check if there is an IAM OIDC provider configured already

eksctl utils associate-iam-oidc-provider --cluster demo-cluster-2 --approve



How to setup alb add on:

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json


aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json


Create IAM Role

eksctl create iamserviceaccount \
  --cluster=demo-cluster-2 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam:: 183510238823:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve


Deploy ALB controller

sudo snap install helm --classic

Add helm repo
helm repo add eks https://aws.github.io/eks-charts
Update the repo

helm repo update eks
Install

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id>

  
Verify that the deployments are running:-
kubectl get deployment -n kube-system aws-load-balancer-controller


kubectl get pods -n game-2048
kubectl get pods -n game-2048 -w
kubectl get ingress -n game-2048
kubectl get ingress -n game-2048
kubectl get deploy -n kube-system

end
