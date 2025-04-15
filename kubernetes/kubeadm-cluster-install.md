# Kubernetes cluster installation with kubeadm
## Install required packages
    sudo apt-get update
    # apt-transport-https may be a dummy package; if so, you can skip that package
    sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

    # If the folder `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
    # sudo mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring

    # This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly
    
    apt update
    apt install -y apt-transport-https
    apt install -y kubelet kubeadm kubectl containerd
    apt-mark hold kubelet kubeadm kubectl containerd

## Containerd config

    cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay
    sudo modprobe br_netfilter

Configure net bridge:

    cat <<EOF | sudo tee /etc/sysctl.d/kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward = 1
    EOF

    sudo sysctl --system

    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    sudo systemctl restart containerd

If you are on ubuntu 22, set SystemdCgroup = true in containerd config:

    nano /etc/containerd/config.toml

## Disable swap

    swapoff -a

comment swap entry in fstab

    nano /etc/fstab

## Init manager node

    sudo kubeadm init

Set KUBECONFIG variable:

    export KUBECONFIG=/etc/kubernetes/admin.conf

    ## add the same line in ~/.bashrc

    nano ~/.bashrc

After the initialization you will be able to expand cluster by adding the worker nodes using the token provided by kubeadm init:

    kubeadm join 192.168.1.135:6443 --token 2hkqxq.fjd3mr77yztzeth2 --discovery-token-ca-cert-hash sha256:fa3429f519ea45a1ceea7fbb676acf5c4c90d44a03d0218a4f2085ad1658af17

## Install pod network:

    curl https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico.yaml -O
    kubectl apply -f calico.yaml

After pod network installation the nodes should be in ready status:

    NAME    STATUS   ROLES           AGE    VERSION
    kube1   Ready    control-plane   121m   v1.25.0
    kube2   Ready    <none>          106m   v1.25.0
