
Welcome to the Kubernets depoly!

## Content
### [1.-Kubernetes-basic-configuration](https://github.com/andrezheng-git/K8s/wiki/1.-Kubernetes-basic-configuration)
### [2.-Kubernetes-demo-deploy](https://github.com/andrezheng-git/K8s/wiki/2.-Kubernetes-demo-deploy)
### [3.-Jenkins-kubernetes-deploy-note](https://github.com/andrezheng-git/K8s/wiki/3.-Jenkins-kubernetes-deploy-note)

Welcome to the K8s wiki!

### Version
***
| Module | version |
| ---- | ---- |
| Kubeadm | v1.16.0 | 
| kubectl | v1.16.0 |
| kube-apiserver | v1.16.0 |
| kube-controller-manager | v1.16.0|
| kube-proxy | v1.16.0 |
| pause | 3.1 |
| etcd | 3.3.15-0 |
| coredns | 1.6.2 |
| Linux kernel | 4.9.220 |
| docker | 19.03.12 |


### Preparation
***
#### 1. Kernel update:a higher kernel version would be better for kubeadm.  
```
$ wget https://cbs.centos.org/kojifiles/packages/kernel/4.9.220/37.el7/x86_64/kernel-4.9.220-37.el7.x86_64.rpm  
$ rpm -ivh kernel-4.9.220-37.el7.x86_64.rpm  
$ reoot  
$ uname -r  
```
#### 2. Close selinux  
```
$ setenforce 0  
$ getenforce
```
#### 3. Close swap portion  
```
$ cat vi /etc/fstab  
```
```
/dev/mapper/centos-home /home                   xfs     defaults        0 0
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/mapper/vgdata-lv01 /data                   xfs     defaults        0 0
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ reboot  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ free -h  

#### 4. Docker install
To accelerate the download speed, docker mirror url could be config as follows:  
```
$ cat /etc/docker/daemon.json   
```
```
{"registry-mirrors": ["https://f4qs770y.mirror.aliyuncs.com"]}
```

#### 5. yum repo config   
```
$ cat /etc/yum.repos.d/kubernetes.repo  
```
```
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

```

#### 6. kubeadm,kubelet,kubectl install   
```
$ yum install -y kubelet-1.16.0 kubeadm-1.16.0 kubectl-1.16.0 --disableexcludes=kubernetes  
$ systemctl enable kubelet && systemctl start kubelet   
```
#### 7. iptalbes config 
```
$ echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables   
$ echo 1 > /proc/sys/net/ipv4/ip_forward    
```
#### 8. kubeadm images pull
To pull the images required manually.
```
$ kubeadm config images list
```
```
k8s.gcr.io/kube-apiserver:v1.16.0
k8s.gcr.io/kube-controller-manager:v1.16.0
k8s.gcr.io/kube-scheduler:v1.16.0
k8s.gcr.io/kube-proxy:v1.16.0
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.15-0
k8s.gcr.io/coredns:1.6.2
```
And use docker to pull the images manually.

### Master install
***
#### 1. Initialization
```
$ kubeadm init --kubernetes-version=1.16.0 --pod-network-cidr=172.17.0.0/16 --apiserver-advertise-address=xx.xx.xx.xx
```
#### 2. flanneld install
```
$ mkdir -p $HOME/.kube  
$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
$ chown $(id -u):$(id -g) $HOME/.kube/config  
```
#### 3. flanneld install
```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml  
$ systemctl restart docker  
$ kubectl get nodes 
```
#### 6. for demo, allow anonymous access
```
$ kubectl create clusterrolebinding test:anonymous --clusterrole=cluster-admin --user=system:anonymous
```


### Node install
***
#### 1. Pull the images manully
```
$ docker pull quay.io/coreos/flannel:v0.12.0-amd64  
$ docker pull k8s.gcr.io/kube-proxy:v1.16.0  
$ docker pull k8s.gcr.io/pause:3.1  
```

#### 2. From master, gain the token for nodes to join.
```
$ kubeadm token create --print-join-command  
```
#### 3. Join node to master
```
$ kubeadm join 192.168.242.138:6443 --token XXXXXXXXXXX  --discovery-token-ca-cert-hash sha256:XXXXXXXXXXXXX
```

### dashboard install
***
#### 1. dashboard install  
```
$ kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
```
#### 2. access url:
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

#### 3. Create admin and binding role for dashboard
```
$ cat admin-user.yaml  
```
```
apiVersion: v1  
 kind: ServiceAccount  
metadata:  
 name: admin-user  
 namespace: kube-system  
```

```
$ kubectl create -f admin-user.yaml
$ cat admin-user-role-binding.yaml
```
```
apiVersion: rbac.authorization.k8s.io/v1beta1  
kind: ClusterRoleBinding  
metadata:  
  name: admin-user  
roleRef:  
  apiGroup: rbac.authorization.k8s.io  
  kind: ClusterRole  
  name: cluster-admin  
subjects:  
- kind: ServiceAccount  
  name: admin-user  
  namespace: kube-system  
```
```
$ kubectl create -f  admin-user-role-binding.yaml
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
Get a token to login.








