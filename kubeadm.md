# Initializing KUBEADM (master)

1. `ssh root@<ip-from-cloud>`

2. `apt-get update && apt-get install -y apt-transport-https curl`

3. `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -`

     `cat <<EOF >/etc/apt/sources.list.d/kubernetes.list`

      `deb https://apt.kubernetes.io/ kubernetes-xenial main`

       `EOF`

4. `apt-get update`

     `apt-get install -y kubelet kubeadm kubectl`

     `apt-mark hold kubelet kubeadm kubectl`

5. `echo "installing docker"`

   `apt-get update`

   `apt-get install -y \`

      ` apt-transport-https \`

      ` ca-certificates \`

      `curl \`

      `software-properties-common`

   `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -`

   `add-apt-repository \`

      `"deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \`

      `$(lsb_release -cs) \`

      `stable"`

   `apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')`


6. `echo "installing kubernetes"`

   `apt-get update && apt-get install -y apt-transport-https`

   `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -`

   `cat <<EOF >/etc/apt/sources.list.d/kubernetes.list`

   `deb http://apt.kubernetes.io/ kubernetes-xenial main`

   `EOF`

   `apt-get update`

   `apt-get install -y kubelet kubeadm kubectl`


7. `echo "deploying kubernetes (with calico)..."`

   `kubeadm init --pod-network-cidr=192.168.0.0/16`

   `export KUBECONFIG=/etc/kubernetes/admin.conf`

   `kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml`

   `kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml`


8. `adduser <username>` --------------------`adduser masud`

   ```
   Set password prompts:
   Enter new UNIX password:
   Retype new UNIX password:
   passwd: password updated successfully
   
   User information prompts:
   Changing the user information for username
   Enter the new value, or press ENTER for the default
       Full Name []:
       Room Number []:
       Work Phone []:
       Home Phone []:
       Other []:
   Is the information correct? [Y/n]
   ```

   `usermod -aG sudo <username>` ----- `usermod -aG masud`

   ` su - <username>` -------------------------`su - masud`


9. `mkdir -p $HOME/.kube`

   `sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`

   `sudo chown $(id -u):$(id -g) $HOME/.kube/config`


10. `kubectl taint nodes --all node-role.kubernetes.io/master-`


11. Then deploy, expose, ingress etc.





# Joining Another Node to Master node

1. First `6` steps are similar as `kubeadm-initializing`

2. `kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>`


   Example :

   ```bash
   kubeadm join 68.183.186.19:6443 --token kbkora.dbjuaempg4wqb61v --discovery-token-ca-cert-hash sha256:757ba455b740b236f9ac50d08e8523eebf5b590e25e7c98cc284442f906e2bb2
   ```

   

#### To get token (On master node)

`kubeadm token list`

##### Generally tokens expire after 24 hours. After Expiration new token needs to be generated (On master node)

`kubeadm token create`

#### To get ca-cert-hash (On master node)

`openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \`

   `openssl dgst -sha256 -hex | sed 's/^.* //'`



# Tearing Down any Node

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

```bash
kubeadm reset
```

```bash
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

```bash
ipvsadm -C  #doesnot work
```



###### If you wish to start over simply run `kubeadm init` or `kubeadm join` with the appropriate arguments

