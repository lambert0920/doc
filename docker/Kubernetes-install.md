## Minikube快速搭建K8S单节点环境
### requirement
- kubectl
- macOS
    + Hyperkit driver, xhyve driver, VirtualBox, or VMware Fusion
- Linux
	+ VirtualBox or KVM
	+ NOTE: Minikube also supports a –vm-driver=none option that runs the Kubernetes components on the host and not in a VM. Docker is required to use this driver but no hypervisor.
- Windows
	- VirtualBox or Hyper-V
- VT-x/AMD-v virtualization must be enabled in BIOS
- Internet connection on first run
### 1.centos 安装
#### 安装kubectl
```
# wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kubectl
# install kubectl /usr/local/bin/
# kubectl cluster-info
```
#### 设置shell命令补全
```
# yum install -y bash-completion
# echo "source <(kubectl completion bash)" >> ~/.bashrc
# kubectl
```
#### 安装Hypervisor(KVM) 或者install virtualbox
```
# yum install qemu-kvm libvirt libvirt-python libguestfs-tools virt-install
# systemctl enable libvirtd && systemctl start libvirtd
# wget https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-centos7
# install docker-machine-driver-kvm-centos7 /usr/local/bin/docker-machine-driver-kvm
# usermod -a -G libvirt $(whoami)
# newgrp libvirt
```
#### 安装minikube
```
# wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
# install minikube-linux-amd64 /usr/local/bin/minikube
# minikube config set WantReportErrorPrompt false
# minikube start --vm-driver=kvm
# kubectl version
# kubectl cluster-info
# kubectl get all
```
#### 测试
```
kubectl get nodes
kubectl get pods
minikube stop
```
### 2. windows 安装
+ requirement
	+ Download the minikube-windows-amd64.exe file, rename it to minikube.exe and add it to your path
	+ kubectl,下载，添加到环境变量
	+ virtualbox
+ 验证安装
```
minikube version
kubectl version
```


### 3. 从阿里云的镜像地址在centos安装minikube
```
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.28.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
minikube start --registry-mirror=https://registry.docker-cn.com  # 缺省Minikube使用VirtualBox驱动来创建Kubernetes本地环境
minikube dashboard    # 打开Kubernetes控制台
```


## Kops install Kubernetes cluster on Amazon AWS
### 安装kops 和 awscli
```
# install some tools
sudo yum install -y vim telnet bind-utils wget
# install kops
wget https://github.com/kubernetes/kops/releases/download/1.10.0/kops-linux-amd64
mv kops-linux-amd64 kops
sudo mv kops /usr/local/bin/
sudo chmod +x /usr/local/bin/kops
# install kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
# install python pip
wget https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
rm -rf get-pip.py
# install aws cli
sudo pip install awscli
```
### aws 准备和设置
```
1. 先创建一个组,kops-group,使用adminxx
2. 先创建一个用户，kops，使用program访问，使用kops-group,现在access key 和secret key
3. aws configure
4. 配置dns，进入route53，注册一个domain，例如： 360pf.net
5. 域名验证
	1. dig +short 360pf.net ns
	2. dig +short 360pf.net soa
6. 创建hosted zones
	1. 创建subdomain   k8s.360pf.net   type: public hosted zone
	2. 在主域名下面创建子域名的 ns记录，类型ns，值就是subdomain的ns记录
	3. 子域名验证
		1. dig +short k8s.360pf.net ns
		2. dig +short k8s.360pf.net soa
7. 创建s3 bucket
	1. bucket name: kops.k8s.360pf.net

```
### 创建
```
1. 创建一个ssh key: tlz_cluster 
2. 创建一个集群的配置，保存到s3
kops create cluster --name=k8s.360pf.net --ssh-public-key=~/.ssh/tlz_cluster.pub --state=s3://kops.k8s.360pf.net --zones=us-west-1a  --node-count=2 --node-size=t2.medium --master-size=t2.medium --dns-zone=k8s.360pf.net
kops create cluster --name=k8s.360pf.net  --state=s3://kops.k8s.360pf.net --zones=us-west-1a  --node-count=2 --node-size=t2.medium --master-size=t2.medium --dns-zone=k8s.360pf.net
### 如果配置信息有问题，可以先删除，然后创建
kops delete cluster --name=k8s.360pf.net  --state=s3://kops.k8s.360pf.net          -- 确认删除配置信息
kops delete cluster --name=k8s.360pf.net  --state=s3://kops.k8s.360pf.net --yes    -- 删除集群
### 真正的创建虚机
kops update cluster k8s.360pf.net --yes   --state=s3://kops.k8s.360pf.net  
## 验证状态
ssh -i ~/.ssh/id_rsa admin@api.k8s.360pf.net                                  # ssh to the master
kubectl get nodes --show-labels                                               # list nodes
kops validate cluster --state=s3://kops.k8s.360pf.net                         # 验证cluster的状态
```
```
systemctl status etcd kube-apiserver kube-controller-manager kube-scheduler | grep "(running)" | wc -l
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo firewall-cmd --state
sudo systemctl status firewalld
```