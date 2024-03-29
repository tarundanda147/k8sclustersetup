# Install Kubernetes with AWS Cloud Provider
Install Kubernetes (k8s) on AWS (Ubuntu 20.04 LTS Server)

# Setting VPC
Create new vpc with **10.0.0.0/16** CIDR

Name | Value
--- | ---
**Name tag** | K8s-cluster-vpc
**IPv4 CIDR block** | 10.0.0.0/16 
**IPv6 CIDR blocK** | No IPv6 CIDR block 
**TenancyInfo** | Default

Righ click, select **Manage tags** and add this tag on the previously created vpc

Key | Value
--- | ---
**Name** | K8s-cluster-vpc
**kubernetes.io/cluster/kubernetes** | owned

Select **Edit DNS hostnames** then enable DNS hostnames

Name | Value
--- | ---
**DNS hostnames** | ✅


# Subnet
Create a subnet for the vpc

Name | Value
--- | ---
**Name tag** | K8s-cluster-net
**VPC\*** | (Select K8s-cluster-vpc)
**Availability Zone** | (Select the desired zone E.g. **ap-southeast-1a**)
**VPC CIDRs** | (Auto generated)
**IPv4 CIDR block\*** | 10.0.0.0/24

Select **Modify auto-assign IP settings** then Enable Public IPs for EC2 instances 

Name | Value
--- | ---
**Auto-assign IPv4** | ✅ Enable auto-assign customer-owned IPv4 address

Select **Add/Edit Tags** then add this tag

Key | Value
--- | ---
**Name** | K8s-cluster-net
**kubernetes.io/cluster/kubernetes** | owned


# Internet Gateway
Create an Internet Gateway to route traffic from the subnet into the Internet

Name | Value
--- | ---
**Name tag** | K8s-cluster-igw

Select **Manage Tags** then add this tag

Key | Value
--- | ---
**Name** | k8s-cluster-igw
**kubernetes.io/cluster/kubernetes** | owned

Select **Attach to VPC** then attach to previously created vpc 

Name | Value
--- | ---
**VPC\*** | (Select K8s-cluster-vpc)


# Route Table
Create a routing tablle

Name | Value
--- | ---
**Name tag** | K8s-cluster-rtb
**VPC\*** | (Select K8s-cluster-vpc)

Select **Add/Edit Tags** then add this tag

Key | Value
--- | ---
**Name** | K8s-cluster-rtb
**kubernetes.io/cluster/kubernetes** | owned

Select **Edit routes** add a new route to the 0.0.0.0/0 network via the Internet Gateway we've created

Destination | Target | Status | Propagated
--- | --- | --- | --- 
10.0.0.0/16 | local | active | No
0.0.0.0/0 | (Select k8s-cluster-igw) | | No

Select **Edit subnet associations** then choose previously created subnet

\# | Subnet ID | IPv4 CIDR | IPv6 CIDR | Current Route Table
--- | --- | --- | --- | ---
✅ | (k8s-cluster-net) | 10.0.0.0/24 | - | Main


# IAM Role
Create IAM EC2 roles for node master and worker to make Kubernetes work on AWS

# IAM Master role
Create new Policy by Go to the **IAM -> Policies -> Create policy** then add this in to the JSON

Policy Name : **k8s-cluster-iam-master-policy**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "ec2:DescribeInstances",
                "ec2:DescribeRegions",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVolumes",
                "ec2:CreateSecurityGroup",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:ModifyInstanceAttribute",
                "ec2:ModifyVolume",
                "ec2:AttachVolume",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:CreateRoute",
                "ec2:DeleteRoute",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteVolume",
                "ec2:DetachVolume",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:DescribeVpcs",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:AttachLoadBalancerToSubnets",
                "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:CreateLoadBalancerPolicy",
                "elasticloadbalancing:CreateLoadBalancerListeners",
                "elasticloadbalancing:ConfigureHealthCheck",
                "elasticloadbalancing:DeleteLoadBalancer",
                "elasticloadbalancing:DeleteLoadBalancerListeners",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeLoadBalancerAttributes",
                "elasticloadbalancing:DetachLoadBalancerFromSubnets",
                "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                "elasticloadbalancing:ModifyLoadBalancerAttributes",
                "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:CreateTargetGroup",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:DeleteTargetGroup",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeLoadBalancerPolicies",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeTargetHealth",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:ModifyTargetGroup",
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
                "iam:CreateServiceLinkedRole",
                "kms:DescribeKey"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Then go to **Roles** create new role with EC2 type
Search previously created policy (**k8s-cluster-iam-master-policy**)

Role Name: **k8s-cluster-iam-master-role**


# IAM Worker role
In the same way, Create new **Policy** then add this in to the JSON

Policy Name : **k8s-cluster-iam-worker-policy**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeRegions",
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:BatchGetImage"
            ],
            "Resource": "*"
        }
    ]
}
```

Then go to **Roles** create new role with EC2 type
Search created policy (**k8s-cluster-iam-worker-policy**)

Role Name: **k8s-cluster-iam-worker-role**


# Running EC2
## Master Node
Create an EC2 Ubuntu Server 20.04 LTS using **t2.medium** type (*Recommended*). Kubernets master need at least 2 CPU cores, if you use Free Tear **t2.micro**, you'll facing error NumCPU on kubeadm init (It **can be ignores** but *Not Recommended*).

**# Choosing AMI**

Select **Ubuntu Server 20.04 LTS**

**# Choose Instance Type**

Select **t2.medium** 

**# Configure Instance Details**

Name | Value
--- | ---
**Network** | select previously created VPC (Select **K8s-cluster-vpc**)
**IAM role** | select previously created master role (Select **k8s-cluster-iam-master-role**)

**# Add Tags**

Key | Value
--- | ---
**Name** | Master
**kubernetes.io/cluster/kubernetes** | owned

**# Configure Security Group**

Type | Protocol | Port range | Source | Description - optional
--- | --- | --- | --- | ---
HTTP | TCP | 80 | 0.0.0.0/0 | Protocol HTTP
HTTPS | TCP | 443 | 0.0.0.0/0 | Protocol HTTPS
Custom TCP | TCP | 6443 | 0.0.0.0/0 | Kubernetes API Server
Custom TCP | TCP | 2379 - 2380 | 0.0.0.0/0 | etcd server client API
Custom TCP | TCP | 10250 | 0.0.0.0/0 | Kubelet API
Custom TCP | TCP | 10251 | 0.0.0.0/0 | kube-scheduler
Custom TCP | TCP | 10252 | 0.0.0.0/0 | kube-controller-manager
Custom TCP | TCP | 10255 | 0.0.0.0/0 | Read-Only Kubelet API
Custom TCP | TCP | 10259 | 0.0.0.0/0 | Schedular
Custom TCP | TCP | 10257 | 0.0.0.0/0 | Controller
All Traffic | All | All | 10.0.0.0/16 | VPC subnet

Review all changes then **Launch**

## Worker Node
While waiting master node, create new EC2 instane in the same way like the master node but using **k8s-cluster-iam-worker-rolee**

**# Choosing AMI**

Select **Ubuntu Server 20.04 LTS**

**# Choose Instance Type**

Select **t2.medium** 

**# Configure Instance Details**

Name | Value
--- | ---
**Network** | select previously created VPC (Select **K8s-cluster-vpc**)
**IAM role** | select previously created master role (Select **k8s-cluster-iam-worker-role**)

**# Add Tags**

Key | Value
--- | ---
**Name** | Worker
**kubernetes.io/cluster/kubernetes** | owned

**# Configure Security Group**

Type | Protocol | Port range | Source | Description - optional
--- | --- | --- | --- | ---
SSH | TCP | 22 | 0.0.0.0/0 | SSH
HTTP | TCP | 80 | 0.0.0.0/0 | Protocol HTTP
HTTPS | TCP | 443 | 0.0.0.0/0 | Protocol HTTPS
Custom TCP | TCP | 10250 | 0.0.0.0/0 | Kubelet API
Custom TCP | TCP | 10255 | 0.0.0.0/0 | Read-Only Kubelet API
Custom TCP | TCP | 30000-32767 | 0.0.0.0/0 | NodePort Services
All Traffic | All | All | 10.0.0.0/16 | VPC subnet

Review all changes then **Launch**

# Config Kubernetes Cluster
## Master Node Setup
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`

hostnamectl set-hostname $(curl -s http://169.254.169.254/latest/meta-data/local-hostname -H "X-aws-ec2-metadata-token: $TOKEN")

```bash

#!/bin/bash

sudo apt update

sudo swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

sudo apt install -y ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.24/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.24/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.24.17-1.1 kubeadm=1.24.17-1.1 kubectl=1.24.17-1.1
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```
```bash
cat << EOF > /etc/kubernetes/aws.yaml
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
networking:
  serviceSubnet: "10.100.0.0/16"
  podSubnet: "10.244.0.0/16"
apiServer:
  extraArgs:
    cloud-provider: "aws"
controllerManager:
  extraArgs:
    cloud-provider: "aws"
EOF

# init cluster
kubeadm init --config /etc/kubernetes/aws.yaml

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
## Worker Node Setup

TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
hostnamectl set-hostname $(curl -s http://169.254.169.254/latest/meta-data/local-hostname -H "X-aws-ec2-metadata-token: $TOKEN")

```bash

#!/bin/bash

sudo apt update

sudo swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

sudo apt install -y ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.24/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.24/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.24.17-1.1 kubeadm=1.24.17-1.1 kubectl=1.24.17-1.1
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```
Replace `/etc/kubernetes/node.yml` config below with your own:

Token: **"i4pna1.8tlp6kcmukr5sian"**

apiServerEndpoint: **"10.0.0.119:6443"**

caCertHashes: **"sha256:c2974f5f46e06df9bddd532ac61617ada82943b09ee914847fd8f15f7b8ff008"**

name: **ip-10-0-0-186.eu-west-3.compute.internal**

```bash
cat << EOF > /etc/kubernetes/node.yml
---
apiVersion: kubeadm.k8s.io/v1beta1
kind: JoinConfiguration
discovery:
  bootstrapToken:
    token: "i4pna1.8tlp6kcmukr5sian"
    apiServerEndpoint: "10.0.0.119:6443"
    caCertHashes:
      - "sha256:c2974f5f46e06df9bddd532ac61617ada82943b09ee914847fd8f15f7b8ff008"
nodeRegistration:
  name: ip-10-0-0-186.eu-west-3.compute.internal
  kubeletExtraArgs:
    cloud-provider: aws
EOF
```

```bash
# join cluster
kubeadm join --config /etc/kubernetes/node.yml

```

## Check Nodes

Go back to master to check the nodes

```bash
# check nodes
kubectl get nodes

```
