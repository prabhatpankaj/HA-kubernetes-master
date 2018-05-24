![alt text](https://raw.githubusercontent.com/prabhatpankaj/HA-kubernetes-master/master/ha-kubernetes.png)

* Let’s assume the following:
```
master 1 address      = 10.0.0.50
master 2 address      = 10.0.0.51
master 3 address      = 10.0.0.52
load balancer address = 10.0.0.200
```

* [setup your servers with the necessary prerequisites](https://kubernetes.io/docs/setup/independent/install-kubeadm/#before-you-begin)

* create functional cluster on three master nodes.

# for 1st master nodes
```
kubeadm token generate
```
* config.yaml for all masster nodes

```
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
etcd:
  endpoints:
  - "http://10.0.0.50:2379"
  - "http://10.0.0.51:2379"
  - "http://10.0.0.52:2379"
apiServerExtraArgs:
  apiserver-count:3
apiServerCertSANs:
- "10.0.0.50"
- "10.0.0.51"
- "10.0.0.52"
- "10.0.0.200"
- "127.0.0.1"
token: "YOUR KUBEADM TOKEN from 1st master nodes"
tokenTTL: "0"

```
* on the first master instance:
```
kubeadm init --config /path/to/config.yaml
```
* copy directory with all of certs and keys in /etc/kubernetes/pki from one master to others 

* Initialize the other masters
```
kubeadm init --config /path/to/config.yaml
```
* Configure an unprivileged user-account and Take a copy of the Kube config to every master node 
```
sudo useradd kubeuser -G sudo -m -s /bin/bash
sudo passwd kubeuser
sudo su kubeuser
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
echo "export KUBECONFIG=$HOME/admin.conf" | tee -a ~/.bashrc
```
* Edit $HOME/admin.conf and change server: 10.0.0.50 to server: 10.0.0.200

* Run
```
kubectl get nodes
```
output should be 
```
NAME         STATUS    ROLES     AGE       VERSION
10.0.0.50    Ready     master    1h        v1.10.2
10.0.0.51    Ready     master    1h        v1.10.2
10.0.0.52    Ready     master    1h        v1.10.2
```
* Join nodes to the cluster
```
kubeadm join --token YOUR_CLUSTER_TOKEN 10.0.0.200:6443 --discovery-token-ca-cert-hash sha256:89870e4215b92262c5093b3f4f6d57be8580c3442ed6c8b00b0b30822c41e5b3

```
