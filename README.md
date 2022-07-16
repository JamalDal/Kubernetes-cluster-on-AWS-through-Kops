
# Kubernetes cluster on AWS through Kops

Select a Linux-amazon AMI machine for this purpose so that we don’t have to install AWS CLI on it. If we are selecting some other VM, we can configure CLI on it with the help of this article.


https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html





## After creating an EC2-instance

Create a role with 

•	AmazonEC2FullAccess

•	AmazonRoute53FullAccess

•	AmazonS3FullAccess

•	IAMFullAccess

And attach this role to this machine.

Now, create a Route53 private hosted zone, if you don’t have public domain name; and for the experiment I made an entry with the name mytestingenvironment.in


## Initially, install kubectl on your VM

```bash
  curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
  chmod +x ./kubectl
  sudo mv ./kubectl /usr/local/bin/kubectl

```

## Now install kops on the machine


```bash
  curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
  chmod +x kops-linux-amd64
  sudo mv kops-linux-amd64 /usr/local/bin/kops
```

## Create an S3 bucket and expose the environment variable

```bash
  aws s3 mb s3://dev.k8s.mytestingenvironment.in
  export KOPS_STATE_STORE=s3://dev.k8s.mytestingenvironment.in
```

### Now, it is important to create the ssh key before creating the cluster
```bash
  ssh-keygen
```

### There are few things to keep in mind while creating a cluster like zone and dns entry

```bash
  kops create cluster --cloud=aws --zones=us-east-1a --name= dev.k8s.mytestingenvironment.in --dns-zone=mytestingenvironment.in --dns private
```

### Finally, you can validate your cluster

```bash
  kops validate cluster
```

### We are done with creating the cluster, now it is time to explore more
#### To check nodes and their details

```bash
  kubectl get nodes -o wide
```

#### To check pods with their details

```bash
  kubectl get pods -o wide
```

### We can deploy Nginx container on K8s
```bash
  kubectl run sample-nginx --image=nginx --replicas=2 --port=80
  kubectl get pods
  kubectl get deployments
```


## If we check the deployment here, there won’t be any because we haven’t deployed one 

```bash
  kubectl get deployment
```
### And if we check the service, there will be a default one, in this case that with ClusterIP deployment

```bash
  kubectl get service
```

## Expose the Nodeport deployment as service, it will give us ClusterIP and a port number. We will use port number to excess the pods

```bash
  kubectl expose deoloyment web-=server --type Nodeport
```

![101](https://user-images.githubusercontent.com/97054844/179118687-b474b445-7619-4335-ba11-c6a00d3f9f66.png)




### Now, if we want our pod to be accessed from outside, we can use LoadBalancer as deployment type.
It will provide us a DNS address, and for more simplification we can add CNAME and route it to our private/public domain name. 

Moreover, load will be distributed among nodes.


```bash
  kubectl expose deoloyment web-=server --type LoadBalancer
  kubectl get service

```


![Screenshot 2022-07-14 at 8 40 25 PM](https://user-images.githubusercontent.com/97054844/179118943-02a46781-88ba-44c6-a78b-f6f56cfebe07.png)



### Finally, we can delete the cluster easily

```bash
  kops delete cluster dev.k8s.mytestingenvironment.in –yes
```


Note: If you find any flaws just avoid them, currently I am learining and making my own documantation.

```bash
  The End. Thank you!
```


### References:

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

https://github.com/ValaxyTech/DevOpsDemos/blob/master/Kubernetes/k8s-setup.md

https://www.howtoforge.com/how-to-setup-a-kubernetes-cluster-on-aws-using-kops/

https://aws.amazon.com/blogs/compute/kubernetes-clusters-aws-kops/

https://kops.sigs.k8s.io/getting_started/aws/


