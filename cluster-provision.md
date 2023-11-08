# Kubernetes Cluster Provision

## Steps
- [Prerequisites](#prerequisites-all-nodes)
- [Install Container Runtime](#install-container-runtime-interface-all-nodes)
- [Install kubeadm, kubelet, kubectl](#install-kubeadm-kubelet-kubectl-all-nodes)
- [Initialize kubeadm on master node](#initialize-kubeadm-on-master-node)
- [Enable CNI for Pod networking](#enable-cni-plugin-for-pod-networking)
- [Join worker nodes to master node](#join-worker-nodes-to-master-node)

### Prerequisites (All Nodes)
- Take at least two Ubuntu (22.04 LTS) VM, and SSH into the nodes
    ```sh
    sudo su # if already not logged in with root user

    apt update
    apt install -y apt-transport-https ca-certificates curl
    ```
- Enable iptables Bridged Traffic
    ```sh
    cat <<EOF | tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    modprobe overlay
    modprobe br_netfilter

    # sysctl params required by setup, params persist across reboots
    cat <<EOF | tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

    # Apply sysctl params without reboot
    sysctl --system
    ```
- Disable swap
    ```sh
    swapoff -a
    (crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
    ```

### Install Container Runtime Interface (All Nodes)
You can install any of the following container runtimes
- CRI-O
- Docker
- containerd

Installation of `CRI-O` and `Docker` will be shown here.

- #### CRI-O

    ```sh
    OS="xUbuntu_22.04"

    VERSION="1.28"

    cat <<EOF | tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
    deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
    EOF
    cat <<EOF | tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
    deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
    EOF
    ```
    ```sh
    curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
    curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
    ```

    ```sh
    apt update
    apt install cri-o cri-o-runc cri-tools -y
    ```

    ```sh
    systemctl daemon-reload
    systemctl enable crio --now
    ```

### Install kubeadm, kubelet, kubectl (All Nodes)
- Install kubeadm, kubelet, kubectl
    ```sh
    curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
    echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list

    apt update
    ```
    ```sh
    apt install -y kubelet=1.28.2-00 kubectl=1.28.2-00 kubeadm=1.28.2-00
    apt-mark hold kubelet kubeadm kubectl
    ```
- Add node IP to `KUBELET_EXTRA_ARGS`
    ```sh
    apt install -y jq
    local_ip="$(ip --json a s | jq -r '.[] | if .ifname == "eth1" then .addr_info[] | if .family == "inet" then .local else empty end else empty end')"
    cat > /etc/default/kubelet << EOF
    KUBELET_EXTRA_ARGS=--node-ip=$local_ip
    EOF
    ```
    Make sure the `node-ip` is set. if not set, it might be because of `ip link`.Check the interface name using the command `ip link show` and replace `eth1` with the correct name in your script. Most probably it's `eth0`
### Initialize kubeadm on master node
- Master Node with Public IP
    ```sh
    IPADDR=$(curl ipinfo.io/ip && echo "") # Make sure it's set. If not, set manually
    NODENAME=$(hostname -s)
    POD_CIDR="192.168.0.0/16"
    ```
    Init `kubeadm`
    ```sh
    kubeadm init --control-plane-endpoint=$IPADDR  --apiserver-cert-extra-sans=$IPADDR  --pod-network-cidr=$POD_CIDR --node-name $NODENAME --ignore-preflight-errors Swap
    ```
- Master Node with Private IP
    ```sh
    IPADDR=$(hostname -I | awk '{print $1}')
    NODENAME=$(hostname -s) # you can set manually, like `master`
    POD_CIDR="192.168.0.0/16"
    ```
    Init `kubeadm`
    ```sh
    kubeadm init --apiserver-advertise-address=$IPADDR  --apiserver-cert-extra-sans=$IPADDR  --pod-network-cidr=$POD_CIDR --node-name $NODENAME --ignore-preflight-errors Swap
    ```
- Copy kubeconfig
    ```sh
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    ```
    If you are not the root user, run the following
    ```sh
    sudo chown $(id-u):$(id -g) $HOME/.kube/config
    ```

- To verify the cluster is running
    ```sh
    kubectl get --raw='/readyz?verbose'
    kubectl cluster-info 
    ```
- Remove taint from master node, if you want to schedule pods here
    ```sh
    kubectl taint nodes --all node-role.kubernetes.io/control-plane-
    ```
### Enable CNI Plugin for Pod Networking
- Install Calico Network Plugin
    ```sh
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

    curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O

    kubectl create -f custom-resources.yaml
    ```

### Join Worker Nodes to Master Node
- In case you didn't save the join command from the master node, run the following command in `master` node to get the join command.
    ```sh
    kubeadm token create --print-join-command
    ```
 - Run the output join command in the worker nodes
    ```sh
    # sample join command
    kubeadm join <ip-address>:<port> --token <token> --discovery-token-ca-cert-hash <discovery-token>
    ```
- Make sure the worker node is joined, by listing nodes
    ```sh
    kubectl get nodes
    ```
