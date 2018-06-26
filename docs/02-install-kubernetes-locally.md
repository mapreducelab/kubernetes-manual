# Install Kubernetes Locally

### Install Kubernetes Master

Log in to the kubernetes master virtual machine and become root.

```
vagrant ssh master01
sudo -s
```

### Install Docker Daemon

Recommended docker version for Kubernetes v1.10.5 is 17.03, but for simplicity, we will use docker that ships with Centos 7 - v1.13.

```
yum install -y docker
systemctl enable docker && systemctl start docker
```

### Install kubeadm, kubelet and kubectl

- `kubeadm`: the command to bootstrap the cluster.

- `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.

- `kubectl`: the command line util to talk to your cluster.

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

> Disabling SELinux by running setenforce 0 is required to allow containers to access the host filesystem, which is required by pod networks for example. You have to do this until SELinux support is improved in the kubelet.

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

> Some users on RHEL/CentOS 7 have reported issues with traffic being routed incorrectly due to iptables being bypassed. You should ensure `net.bridge.bridge-nf-call-iptables` is set to 1 in your `sysctl` config, e.g.

### Configure cgroup driver used by kubelet on Master Node

Make sure that the cgroup driver used by kubelet is the same as the one used by Docker. Verify that your Docker cgroup driver matches the kubelet config:

```
docker info | grep -i cgroup
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

If the Docker cgroup driver and the kubelet config don’t match, change the kubelet config to match the Docker cgroup driver. The flag you need to change is `--cgroup-driver`. If it’s already set, you can update like so:

```
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Otherwise, you will need to open the systemd file and add the flag to an existing environment line.
Then restart kubelet:

```
systemctl daemon-reload
systemctl restart kubelet
```

### Disable swap memory

Swap disabled. You MUST disable swap in order for kubelet to work properly.

```
swapoff -a
```

### Initialize Kubernetes Cluster (For master only)

```
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<host ip>
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

> output

```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.42.42:6443 --token 3fmgfv.c1xsly03duok3kd2 --discovery-token-ca-cert-hash sha256:9ceed51de6d36d6c470286230c77c306809b6b1283853fd875cc1d1cbcfbb5a3
```

### Deploy the Kubernetes Worker Node

Folow the steps above, EXCEPT for the last "Initialize Kubernetes Cluster" step. Instead, when the Kubernetes worker node is ready, run the join command.

```
kubeadm join 192.168.42.42:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

> Verify kubernetes cluster by running `kubectl get nodes` command.
