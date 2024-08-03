# install k8s on ubuntu

- Disable swap

```shell
  sudo swapoff -a
  sudo sed -i '/swap/s/^/#/' /etc/fstab
```

- Check swap this command should return nothing

```shell
sudo swapon --show
```

- Change host name optional
    - for all servers

```shell
sudo hostnamectl set-hostname :name
```

- Map the hostname to the ip address in the `etc/hosts` file
    - for all servers

```shell
sudo vim /etc/hosts
# :ip :name
```

- Enable IPv4 packet forwarding
    - overlay Module
        - An overlay filesystem combines two filesystems - an 'upper' filesystem and a 'lower' filesystem.
          [source](https://docs.kernel.org/filesystems/overlayfs.html)
        - In the context of Linux containers, an overlay file system is used to layer the changes made by a container on
          top
          of a base image while preserving the original image intact. This allows containers to share a common base
          image
          and reduces the size of the images that need to be transferred, stored, and deployed.
    - [br_netfilter](https://unix.stackexchange.com/questions/756898/what-exactly-does-the-br-netfilter-kernel-module-control)

```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter


cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF


sudo sysctl --system
```

- Install kubelet, kubeadm, and kubectl on each
  node [source](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime)

```shell
sudo apt-get update -y && apt-get upgrade -y
sudo apt-get install -y apt-transport-https ca-certificates curl
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

```

- [install docker](https://docs.docker.com/engine/install/ubuntu/)

- Configure cGroup

```shell
sudo sh -c "containerd config default > /etc/containerd/config.toml"
sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml

sudo apt-get install kubeadm kubelet kubectl
sudo apt-mark hold kubeadm kubelet kubectl
sudo systemctl restart containerd.service
```

- Initialize the Kubernetes cluster on the master node

```shell
sudo kubeadm config images pull
sudo kubeadm init --pod-network-cidr=10.10.0.0/16


# Your Kubernetes control-plane has initialized successfully!

# To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf
# Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 49.13.20.112:6443 --token z95sbc.8h73yb7vojxftn5h --discovery-token-ca-cert-hash sha256:26f13c47e629e53b2b79513ab6ee14343522e6003d75ca4d61315640cf219713
```

[toto](https://medium.com/@supportfly/how-to-install-kubernetes-on-ubuntu-11c2901fb659)
kubernetes-dashboard   kubernetes-dashboard-kong-proxy 443
[ports](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)
