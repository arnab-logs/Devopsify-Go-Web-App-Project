# Install EKS

Please follow the prerequisites doc before this step.

## Install a EKS cluster with EKSCTL

```

eksctl create cluster --name demo-cluster --region ap-south-1 --node-type c7i-flex.large --nodes-min 2 --nodes-max 2

```

## Delete the cluster

```
eksctl delete cluster --name demo-cluster --region ap-south-1
```