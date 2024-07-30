# Setting up K8s Cluster using LXC/LXD
> **Note:** For development purpose and not recommended for Production use

## Prerequisites
LXC need to be installed to successfully deploy the Kubernetes cluster. That is part of the bootstrap script.

> **Note:** The naming convention is k8s master node name has to have **master** keyword in the name and for k8s worker nodes **worker** keyword in the name.
```
$ lxc list
+---------+---------+--------------------------------+-----------------------------------------------+-----------+-----------+
|  NAME   |  STATE  |              IPV4              |                     IPV6                      |   TYPE    | SNAPSHOTS |
+---------+---------+--------------------------------+-----------------------------------------------+-----------+-----------+
| master  | RUNNING | 192.168.219.64 (vxlan.calico)  | fd42:a2b4:fed2:a8dc:216:3eff:fe24:c720 (eth0) | CONTAINER | 0         |
|         |         | 10.147.62.238 (eth0)           |                                               |           |           |
+---------+---------+--------------------------------+-----------------------------------------------+-----------+-----------+
| worker1 | RUNNING | 192.168.235.128 (vxlan.calico) | fd42:a2b4:fed2:a8dc:216:3eff:fe70:2716 (eth0) | CONTAINER | 0         |
|         |         | 10.147.62.72 (eth0)            |                                               |           |           |
+---------+---------+--------------------------------+-----------------------------------------------+-----------+-----------+
| worker2 | RUNNING | 192.168.189.64 (vxlan.calico)  | fd42:a2b4:fed2:a8dc:216:3eff:fe57:b299 (eth0) | CONTAINER | 0         |
|         |         | 10.147.62.28 (eth0)            |                                               |           |           |
+---------+---------+--------------------------------+-----------------------------------------------+-----------+-----------+
```
## Sysctl setting on host linux machine
**IMPORTANT**:

In order to sucessfully complete the deployment of the Kubernetes cluster the Linux host where you are running lxd containers need to have `nf_conntrack_max` value properly set. Otherwise kube-proxy pods will fail.
```
sysctl -w net/netfilter/nf_conntrack_max=131072
```

Above is covered by the bootstrap script.

## Instalation

```sh
$ git clone https://github.com/mariano-italiano/lxd-k8s.git
$ cd lxd-k8s
$ ./kubelx provision
```

## Verify installation
### Exec into kmaster node
```
$ lxc exec master bash
```
### Verifying Nodes
```
$ kubectl get nodes
NAME      STATUS   ROLES           AGE    VERSION
master    Ready    control-plane   3m8s   v1.29.7
worker1   Ready    <none>          117s   v1.29.7
worker2   Ready    <none>          45s    v1.29.7
```

### Verifying cluster version
```
$ kubectl cluster-info
Kubernetes master is running at https://10.127.221.187:6443
KubeDNS is running at https://10.127.221.187:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

## To access k8s cluster without execing into kmaster node

### Download and install the kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
echo 'source <(kubectl completion bash)' >>~/.bashrc
source ~/.bashrc
```
### Create .kube directory
```
$ mkdir ~/.kube
```
### Copy config from master into .kube directory
```
$ lxc file pull master/etc/kubernetes/admin.conf ~/.kube/config
$ ls -l ~/.kube
```
### Try to access k8s cluster without execing into kmaster node
```
$ kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
kmaster     Ready    master   23m   v1.19.2
kworker01   Ready    <none>   19m   v1.19.2
kworker02   Ready    <none>   17m   v1.19.2
```
