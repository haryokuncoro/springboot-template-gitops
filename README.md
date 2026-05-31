## Login to argocd from command line.
 ```
argocd login argocd.haryokuncoro.xyz --username admin
```


# cluster name
platform-dev-eks

# Add Repository in argocd.
```
argocd repo add git@github.com:haryokuncoro/springboot-template-gitops.git --ssh-private-key-path ~/.ssh/github
```

## Attaching the IAM policy to the node group for ECR Access.
Find the node group name.
```
aws eks list-nodegroups \
  --cluster-name platform-dev-eks \
  --region us-east-1
```

## Find the ROLE name for the node group name.
```
aws eks describe-nodegroup   --cluster-name platform-dev-eks   --nodegroup-name platform-dev-eks-ng   --region us-east-1   --query "nodegroup.nodeRole"   --output 
```

## Attach ECR read policy to the role
```
aws iam attach-role-policy \
  --role-name platform-dev-eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
```

## List the attached policies
```
aws iam list-attached-role-policies \
  --role-name platform-dev-eks-node-role \
  --output table
```


## Create folder argocd in that create apps and projects folder.
cd argocd
kubectl apply -f projects/springboot-project.yaml
kubectl apply -f apps/springboot-project.yaml
