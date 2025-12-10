# GitOps: Multi cluster hube-spoke deployment using ArgoCD

This guide sets up a hub-and-spoke GitOps setup on EKS using ArgoCD. You’ll install the required CLI tools, create one hub cluster and two spoke clusters, expose the ArgoCD UI, and connect each spoke to the hub for centralized deployment management.

---

## Step 1: prerequisite
- install kubectl: https://kubernetes.io/docs/tasks/tools/
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

- install aws cli: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
	- Remove previous version: `sudo apt remove awscli`
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

- check version: `aws version --client`
- configure aws cli into local machine: run `aws configure` 
- access key, access ID:  go-to-aws security credentials and create access key. 
	
- install eksctl into local machine:  https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html#eksctl-install-update
	- Install eksctl:
```
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```
- check version: `eksctl version`
	
- Install argocd into your terminal
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

- Install argoCD CLI: 
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd 
rm argocd-linux-amd64
```
- login to argoCD in CLI: `argocd login <hub_host_ip>:30080`


## Step 2: create cluster
- Create one hub cluster and 2 spoke cluster  
- hub-cluster: `eksctl create cluster --name hub --region us-east-2`
- spoke-1: `eksctl create cluster --name spoke-1 --region us-east-2`
- spoke-2: `eksctl create cluster --name spoke-2 --region us-east-2`


## Step 3: login to argoCD UI
- login to argoCD UI : 
			- run `kubectl edit svc/argocd-server -n argocd`
			- change service type to NodePort 
			- Set `nodePort: 30080` port number from 30000 - 32767
			- ArgoCD UI: `https://<hub_host_ip>:30080`
			- allow port in SG inbound rules

  
## Step 4: connect spoke cluster with hub

- check cluster: `kubectl config get-contexts`
```
argocd cluster add <spoke-cluster-context> --server <hub_host_ip>:30080
```
