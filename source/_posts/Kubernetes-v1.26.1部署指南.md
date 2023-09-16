---
title: Kubernetes-v1.26.1部署指南
date: 2023-09-01 09:27:41
tags: Kubernetes
---

## 1 安装k8s前系统准备

1. 关闭防火墙

```bash
#临时关闭
$ systemctl stop firewalld
#永久关闭
$ systemctl disable firewalld

```

2. 关闭selinux

```bash
#永久关闭
$ sed -i '/selinux/s/enforcing/disabled/' /etc/selinux/config
#临时关闭
$ setenforce 0
```

3. 关闭swap

```bash
#临时关闭
$ swapoff -a
#永久关闭
$ sed -ri 's/.*swap.*/#&/' /etc/fstab
```

4. 将桥接的 IPv4 流量传递到 iptables 的链(所有节点都设置)

```bash

#启用必要的模块
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

$ cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

#配置生效
$ sudo sysctl --system
```

5. 安装`containerd`

```bash

$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

$ sudo dnf update
$ sudo dnf install -y containerd
$ sudo mkdir -p /etc/containerd
$ sudo containerd config default | sudo tee /etc/containerd/config.toml
# 修改containerd配置

$ sudo vi /etc/containerd/config.toml

# 1、找到[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]并将值更改SystemdCgroup为true
# 2、找到sandbox_image = "k8s.gcr.io/pause:3.6"并改为sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.6"

# 重启以应用更改
$ sudo systemctl restart containerd
# 加入开机启动
$ sudo systemctl enable containerd
```

6. 修改`/etc/hosts`配置主机与IP地址映射，避免以后IP变化导致k8s集群大量适配工作

```bash
10.211.55.7 baihl-node
10.211.55.6 baihl-master
```

## 2 安装k8s集群和网络插件Flannel

以下步骤没有显示说明在指定节点上执行的步骤，则在所有节点上执行。

**1、添加kubernetes仓库**

```shell

$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

```

**2、安装Kubernetes modules**

```shell

# 当前最新版本v1.26.1

$ sudo dnf update
$ sudo dnf install -y kubelet-1.26.1 kubeadm-1.26.1 kubectl-1.26.1

# 加入开机启动
$ sudo systemctl enable kubele
```

**3、部署kubernetes集群**

(1)、在k8s-master节点上初始化集群

```shell
# pod-network-cidr填写地址与后面安装的flannel插件默认的网段保持一致
$ kubeadm init   \
	--apiserver-advertise-address=10.211.55.6   \
	--image-repository registry.aliyuncs.com/google_containers   \
	--pod-network-cidr=10.244.0.0/16   \
	--control-plane-endpoint=baihl-master \
	--kubernetes-version=1.26.1
# 等待一会，成功初始化后，输出内容如下：

-------------------------------------

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:

https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join baihl-master:6443 --token uh9zuw.gy0m40a90sd4o3kl \
	--discovery-token-ca-cert-hash sha256:24490dd585768bc80eb9943432d6beadb3df40c9865e9cff03659943b57585b2

-------------------------------------
```

kubeadm join从输出的末尾复制命令并将其保存在安全的地方。稍后我们将使用此命令来允许工作节点加入集群。如果您忘记复制该命令，或者找不到它，您可以使用以下命令重新生成它：

```shell
$ sudo kubeadm token create --print-join-command
```

接下来按照输出提示创建和声明目录

```shell

$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

(2)、将pod网络部署到集群

```shell

$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

-------------------------------------

podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

-------------------------------------

```

验证主节点现在是否准备就绪：

```shell

$ sudo kubectl get nodes
-------------------------------------
NAME STATUS ROLES AGE VERSION
k8s-master Ready control-plane 2m50s v1.26.1
-------------------------------------

```

(3)、在子节点（k8s-node01、k8s-node02）按照上面的提示添加工作节点  
运行第（1）步kubeadm join的命令，将节点加入集群：
```shell
$ kubeadm join baihl-master:6443 --token civoo3.tmx93chp1ejoud38 --discovery-token-ca-cert-hash sha256:1e5a869172dd6934a057b1e5f75f09d5f58bfbc6418c7294b08b3b8880998fb5

[preflight] Running pre-flight checks

[preflight] Reading configuration from the cluster...

[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"

[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"

[kubelet-start] Starting the kubelet

[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:

* Certificate signing request was sent to apiserver and a response was received.

* The Kubelet was informed of the new secure connection details.
  

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
两个节点加入完成后，在master节点上验证如下：
```shell
$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
baihl-master   Ready    control-plane   17h   v1.26.1
baihl-node     Ready    <none>          16h   v1.26.1
```
可以将子节点设置为worker角色：
```
$ kubectl label node baihl-node node-role.kubernetes.io/worker=worker
node/baihl-node labeled

$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
baihl-master   Ready    control-plane   17h   v1.26.1
baihl-node     Ready    worker          16h   v1.26.1
```


## 3 安装Dashboard
**1. 安装Dashboard服务**
此处使用helm进行安装，参考官网：[https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard](https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard)
```bash
# Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

**2、创建用户：**

```shell
$ wget https://raw.githubusercontent.com/cby-chen/Kubernetes/main/yaml/dashboard-user.yaml
$ kubectl apply -f dashboard-user.yaml
```

**3、创建登录Token：**

```shell
$ kubectl -n kubernetes-dashboard create token admin-user
-------------------------------------
eyJhbGciOiJSUzI1NiIsImtpZCI6IkQ1aFlySU9EYzBORlFiZkxLUU5KN0hFRlJZMXNjcUtSeUZoVHFnMW1UU00ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjc2NjI0ODM3LCJpYXQiOjE2NzY2MjEyMzcsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiZjk0MjU5MGItZjgzNC00ZDVkLTlhZGItNmI0NzY0MjAyNmUzIn19LCJuYmYiOjE2NzY2MjEyMzcsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.eWxD-pVzY9S-QcS4r-YpY7MAzZMg0jgP_Dj0i64aH8z2_NU25IJuNYHWB-3A7H6oEMEAofSbIYui-uE8a2oroLylwSPPP_IjcKmGZ2AUiFOfSD_R2QXzl2AC5-BsXBK068KzSYBfieesB-oWQjS8hKd4AOHjLKWWZlp9gJd_qdc8BbQWrKlKpmdmczQvXpeufj371W_taJIH_xxogmUVMgJOVxwawNsD5YGt0O7-_Y70s8AL9DQs3fAAU4YXGG8TmOI3yvOQCqNgfZuiVg2uE5dc4SGzk_FfBOf3QNCpcL1tvjKe6mH5GWlCNEYbJ4eu9flny9a4iRR2gGpt30AA5Q
-------------------------------------
```

## 4 其他
1. 设置`SystemdCgroup = true`，否则会出现POD重启

参考： [k8s集群部署时etcd容器不停重启问题及处理](https://blog.csdn.net/ldjjbzh626/article/details/128400797)

2. 排查日志

API Server 的日志文件通常位于 /var/log/kube-apiserver.log 或 /var/log/pods/kube-apiserver-pod-id/kube-apiserver.log。

3. 执行kubeadm init后如果需要再次执行，需要执行一下 kubeadm reset
4. 常用命令
```bash
# 查看所有命名空间下image运行起来的资源
kubectl get all -o wide -A
# 查看k8s集群所有资源的kind种类
# -o 表示输出 -o name 表示仅仅输出名称
kubectl api-resources --verbs=list --namespaced -o name
```
5. 更换k8s版本
```bash
# 先删除当前装的各个软件
$ dnf remove -y kubelet kubeadm  kubectl

# 安装指定版本
$ dnf install -y kubelet-1.26.1 kubeadm-1.26.1 kubectl-1.26.1
```

6. 通过crictl查看容器、拉镜像等操作\
```shell
# 因为我这里安装的containerd，需要配置crictl的容器云运行时
$ crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
# 执行后生成的配置文件：/etc/crictl.yaml
$ crictl images
-------------------------------------
IMAGE                                                             TAG                 IMAGE ID            SIZE
docker.io/flannel/flannel-cni-plugin                              v1.1.2              7a2dcab94698c       3.84MB
docker.io/flannel/flannel                                         v0.21.2             7b7f3acab868d       24.2MB
-------------------------------------
其他操作：拉取镜像（crictl pull <image>）、查看容器(crictl ps) 等自行百度。
```

7. 网络IP修改操作：[https://technixleo.com/configure-static-ip-address-rhel-centos/](https://technixleo.com/configure-static-ip-address-rhel-centos/)  
8. 生成特定权限和配合的kubeconfig：[https://hex108.gitbook.io/kubernetes-notes/sheng-chan-li-xiao-gong-ju/generate-kubeconfig](https://hex108.gitbook.io/kubernetes-notes/sheng-chan-li-xiao-gong-ju/generate-kubeconfig)

> 本文转载：[https://www.cnblogs.com/zengqinglei/p/17131046.html](https://www.cnblogs.com/zengqinglei/p/17131046.html)