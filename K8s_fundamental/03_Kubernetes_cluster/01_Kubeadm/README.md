## Kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### Installing kubeadm, kubelet and kubectl (1.17.0)

```
export KUBE_VERSION="1.17.0"

sudo apt-get update && sudo apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubeadm=${KUBE_VERSION}-00 kubelet=${KUBE_VERSION}-00 kubectl=${KUBE_VERSION}-00
```

Initialize Kubernetes Cluster

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Copy config

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install CNI (Flannel has been removed from the official after 1.18, if it is used as a test, I think it can still be used)

Continue to Flannel

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```

### Kubectl 
https://kubernetes.io/docs/setup/#learning-environment

Command

```
$ kubectl -n kube-system get pods
NAME                              READY   STATUS    RESTARTS   AGE
coredns-6955765f44-bq644          1/1     Running   0          4m3s
coredns-6955765f44-pvmnh          1/1     Running   0          4m3s
etcd-k8s-dev                      1/1     Running   0          4m
kube-apiserver-k8s-dev            1/1     Running   0          4m
kube-controller-manager-k8s-dev   1/1     Running   0          4m
kube-flannel-ds-amd64-9j86t       1/1     Running   0          3m26s
kube-proxy-rfs7h                  1/1     Running   0          4m2s
kube-scheduler-k8s-dev            1/1     Running   0          4m

$ sudo docker ps

$ cd /etc/kubernetes/manifests
$ ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

$ kubectl cluster-info
Kubernetes master is running at https://10.0.2.15:6443
KubeDNS is running at https://10.0.2.15:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl -n kube-system get pods kube-apiserver-k8s-dev
NAME                     READY   STATUS    RESTARTS   AGE
kube-apiserver-k8s-dev   1/1     Running   0          155m

$ kubectl -n kube-system get pods kube-apiserver-k8s-dev -o=wide
NAME                     READY   STATUS    RESTARTS   AGE    IP          NODE      NOMINATED NODE   READINESS GATES
kube-apiserver-k8s-dev   1/1     Running   0          155m   10.0.2.15   k8s-dev   <none>           <none>

$ kubectl -n kube-system get pods -o=wide
NAME                              READY   STATUS    RESTARTS   AGE    IP           NODE      NOMINATED NODE   READINESS GATES
coredns-6955765f44-bq644          1/1     Running   0          156m   10.244.0.2   k8s-dev   <none>           <none>
coredns-6955765f44-pvmnh          1/1     Running   0          156m   10.244.0.3   k8s-dev   <none>           <none>
etcd-k8s-dev                      1/1     Running   0          156m   10.0.2.15    k8s-dev   <none>           <none>
kube-apiserver-k8s-dev            1/1     Running   0          156m   10.0.2.15    k8s-dev   <none>           <none>
kube-controller-manager-k8s-dev   1/1     Running   0          156m   10.0.2.15    k8s-dev   <none>           <none>
kube-flannel-ds-amd64-9j86t       1/1     Running   0          155m   10.0.2.15    k8s-dev   <none>           <none>
kube-proxy-rfs7h                  1/1     Running   0          156m   10.0.2.15    k8s-dev   <none>           <none>
kube-scheduler-k8s-dev            1/1     Running   0          156m   10.0.2.15    k8s-dev   <none>           <none>

$ kubectl -n kube-system describe pods

$ kubectl -n kube-system describe pods kube-apiserver-k8s-dev

$ kubectl -n kube-system describe nodes k8s-dev

# watch
$ kubectl -n kube-system get pods -w

# cluster info
kubectl version
kubectl cluster-info
kubectl top node
kubectl top pod
kubectl api-versions

kubectl -n kube-system logs kube-apiserver-k8s-dev
# follow mode
kubectl -n kube-system logs -f kube-apiserver-k8s-dev

# copy file host<-->container
kubectl -n kube-system cp

# see difference
vimdiff

kubectl -n kube-system exec kube-apiserver-k8s-dev id
kubectl -n kube-system exec -it kube-apiserver-k8s-dev sh

# bind localhost's port 53 to pod's port 53
sudo kubectl -n kube-system port-forward pod/coredns-6955765f44-bq644 53:53

# first coredns pod
sudo kubectl -n kube-system port-forward pod/`kubectl -n kube-system get pods -o jsonpath="{.items[*].metadata.name}" | tr " " "\n" | grep coredns | head -n1` 53:53

telnet localhost 53
```
