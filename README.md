Utilizando ekclt
- Instalando
 - https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
brew upgrade eksctl && brew link --overwrite eksctl
eksctl version
```

Criando cluster
- https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
- https://eksctl.io/usage/creating-and-managing-clusters/
```shell
eksctl create cluster --name k8s-desafio --region us-east-1
kubectl get nodes -o wide
kubectl get pods -A -o wide
```
- Criando configurando detalhes
```yml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: meu-cluster
  region: us-east-1

vpc:
  subnets:
    public:
      us-east-1a: { id: subnet-0ba469906a3af5f1d }
      us-east-1b: { id: subnet-0daf196973328ac81 }
      us-east-1c: { id: subnet-08cf400a49fb1b679 }

nodeGroups:
  - name: node-group-meu-clluster
    instanceType: t2.medium
    volumeSize: 30
    desiredCapacity: 3 # quantidadede maquinas
    ssh:
      # allow: true
      publicKeyPath: ~/.ssh/id_rsa.pub # chave ssh utilizará:  ~/.ssh/id_rsa.pub
      
```
```shell
eksctl create cluster -f cluster.yaml # cria o cluster
kubectl get nodes -o wide
kubectl get pods -A -o wide
eksctl delete cluster -f cluster.yaml # apaga o cluster
```

Apagar Cluster
- https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
```shell
eksctl delete cluster --name k8s-desafio --region us-east-1
```

Utilizando console awscli
- https://docs.aws.amazon.com/cli/latest/reference/eks/create-cluster.html

Criando cluster
```shell
aws eks create-cluster --name desafio-k8s --region us-east-1 --role-arn arn:aws:iam::763818760783:role/eksClusterRole2 --resources-vpc-config subnetIds=subnet-ef5b8fa6,subnet-1fa24744,subnet-df562fba,securityGroupIds=sg-0bfc78ff0cd8590be

rm -rf ~/.kube/
aws eks update-kubeconfig --name desafio-k8s --region us-east-1
```

Apagar Cluster
```shell
aws eks delete-cluster --name desafio-k8s
```

Criar os worknodes
- Entrar no cloud formation
- https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/template

Colocar o script 
- Amazon S3 URL:
- https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-12-10/amazon-eks-nodegroup.yaml

Nome Cloud Formation: 
- k8s-nodes

Nome cluster
- desafio-k8s

ClusterControlPlaneSecurityGroup
 - Selecione o criado pelo EKS

NodeGroupName
- < UM NOME QUE VC QUEIRA >

NodeImageId
- https://cloud-images.ubuntu.com/docs/aws/eks/
- ami-00ff481e776f6a0c2

KeyName
- < SUA CHAVE SSH CADASTRADA NA AWS >

Selecione a rede
- VPC
- subnets

Nós criados agora é somente ir no seu console e atrelar os nodes ao seu k8s
```shell
wget https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-08-30/aws-auth-cm.yaml
```
No cloudformation criado ir em outputs e copiar o NodeInstanceRole:
- exemplo: arn:aws:iam::473247640396:role/k8s-nodes-NodeInstanceRole-126RT2GZ42G5X
- https://us-east-1.console.aws.amazon.com/cloudformation/

Depois colar no arquivo baixado:
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::xxxxxx:role/k8s-nodes-NodeInstanceRole-xxxxxxxxxx
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:node
```
  
Depois rodar no terminal 
```shell
kubectl apply -f aws-auth-cm.yaml
