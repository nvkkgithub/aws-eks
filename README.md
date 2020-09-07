# AWS EKS Integration

## Create cluster via 'eksctl'
* Assumption: Already VPC, Subnets already created. If not available then use '01-eksctl/cluster-default.yaml' inplace of '01-eksctl/cluster-with-existing-vpc.yaml'

* Install 'eksctl' from https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
```
$ brew tap weaveworks/tap
$ brew install weaveworks/tap/eksctl
$ eksctl version
```

* Update the vpc, subnet values from 'cluster-with-existing-vpc.yaml' file
```
vpc:
  id: "vpc-111111"  
  subnets:
    private:
      eu-west-1a: { id: subnet-111111 }
      eu-west-1b: { id: subnet-222222 }
```

* Create cluster
```
$ eksctl create cluster -f 01-eksctl/cluster-with-existing-vpc.yaml
$ kubectl get nodes
```

* Create fist deployment and expose
```
$ kubectl create deployment hello-deployment --image=k8s.gcr.io/echoserver:1.4

$ kubectl expose deployment hello-deployment --type=LoadBalancer --port=80 --target-port=8080

$ kubectl get svc -o wide
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE   SELECTOR
hello-deployment   LoadBalancer   172.20.48.86   <pending>     80:32168/TCP   50s   app=hello-deployment

```

* Login to pods to debug
```
$ kubectl get po
$ kubectl exec -it hello-deployment-554cb59d45-825t4 -- bash
```


## Create Cluster via 'AWS Console'
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html

### Create IAM cluster role
* Create 'cloud formation template'
```
Refer '02-aws-console/01-CloudFormation.yaml'
```
* Goto AWS Cloudformation -> Create Stack -> Upload above template file.
* Stackname 'eksClusterRole'

### Create Private/Public VPCs
* Use below cloud-formation template
```
https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-08-12/amazon-eks-fully-private-vpc.yaml 
```

### Create cluster
* Step1: Create cluster from 'https://console.aws.amazon.com/eks/home#/clusters' chose 'cluster IAM role' and 'VPCs' from above step.
* Step2: Update kubeconfig for accessing cluster via 'kubectl'
```
$ aws eks --region us-west-2 update-kubeconfig --name cluster_name
$ kubectl get svc

```