#### Kubeadm
#### Создание нод для кластера
В GCP создайте 4 ноды с образом Ubuntu 18.04 LTS:
+ master - 1 экземпляр (n1-standard-2)
+ worker - 3 экземпляра (n1-standard-1)

Закидываем свой ssh-key (проще всего Compute Engine - Metadata - SSH Keys)

```
gcloud compute instances create master --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-2 

gcloud compute instances create node-1 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-1

gcloud compute instances create node-2 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-1 

gcloud compute instances create node-3 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-1

# Для удаления
gcloud compute instances delete master --zone europe-west1-b
```
#### Применяем роль (не хочу руками проходить, заранее извиняюсь просто за shell)
```
cd kubernetes-production-clusters/
ansible-playbook -i inv config.yaml -v
```

#### Создание k8s кластера
на master выполняем
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/24
kubeadm join 10.132.0.24:6443 --token at8hef.4tkp6fbh8qe8or4g \
    --discovery-token-ca-cert-hash sha256:1d55a5c282a3e66a4434a7deb3458a4d839422ae2d7756d77f4d307ecb597e5c
Сохраняем его

# Смотрим
kubectl get node
NAME     STATUS     ROLES    AGE   VERSION
master   NotReady   master   72s   v1.17.4
```

Установим сетевой плагин calico
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Подключаем worker node
```
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-certhash
sha256:<hash></hash></master-port></master-ip></token>


sudo kubeadm join 10.132.0.24:6443 --token at8hef.4tkp6fbh8qe8or4g \
    --discovery-token-ca-cert-hash sha256:1d55a5c282a3e66a4434a7deb3458a4d839422ae2d7756d77f4d307ecb597e5c
```

Проверяем
```
master:~$ kubectl get node
NAME     STATUS     ROLES    AGE   VERSION
master   Ready      master   20m   v1.17.4
node-1   Ready      <none>   76s   v1.17.4
node-2   Ready      <none>   29s   v1.17.4
node-3   NotReady   <none>   4s    v1.17.4
```

Запуск нагрузки

deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2
        ports:
        - containerPort: 80
```
```
kubectl apply -f deployment.yaml
```

#### Обновление мастера

Допускается, отставание версий worker-нод от master, но не наоборот. Поэтому обновление будем начинать с нее - master-ноды

```
sudo apt-get update && apt-get install -y kubeadm=1.18.0-00 \
    kubelet=1.18.0-00 kubectl=1.18.0-00
```    
```
kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
master   Ready    master   26m     v1.18.0
node-1   Ready    <none>   7m37s   v1.17.4
node-2   Ready    <none>   6m50s   v1.17.4
node-3   Ready    <none>   6m25s   v1.17.4
```
```
kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:56:30Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
```
```
kubelet --version
Kubernetes v1.18.0
```

```
kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:58:59Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.13", GitCommit:"30d651da517185653e34e7ab99a792be6a3d9495", GitTreeState:"clean", BuildDate:"2020-10-15T00:59:17Z", GoVersion:"go1.13.15", Compiler:"gc", Platform:"linux/amd64"}
```

```
kubectl describe pod kube-apiserver-master -n kube-system
Name:                 kube-apiserver-master
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 master/10.132.0.24
Start Time:           Wed, 11 Nov 2020 18:09:47 +0000
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: 6b10b75d8da781d8318b7aea4ccdebf1
                      kubernetes.io/config.mirror: 6b10b75d8da781d8318b7aea4ccdebf1
                      kubernetes.io/config.seen: 2020-11-11T18:09:45.584897427Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   10.132.0.24
IPs:
  IP:           10.132.0.24
Controlled By:  Node/master
Containers:
  kube-apiserver:
    Container ID:  docker://fe36ef95aade7296c2e6570e3e4af74c5ea9682bd5b36e814a9edbbfcab503dc
    Image:         k8s.gcr.io/kube-apiserver:v1.17.13
    Image ID:      docker-pullable://k8s.gcr.io/kube-apiserver@sha256:ffdc2f826e3de98608d7e9b41ca0015ec45f066687491142d18a2879fb4a0c22
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=10.132.0.24
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
      --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
      --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
      --etcd-servers=https://127.0.0.1:2379
      --insecure-port=0
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=6443
      --service-account-key-file=/etc/kubernetes/pki/sa.pub
      --service-cluster-ip-range=10.96.0.0/12
      --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    State:          Running
      Started:      Wed, 11 Nov 2020 18:09:50 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        250m
    Liveness:     http-get https://10.132.0.24:6443/healthz delay=15s timeout=15s period=10s #success=1 #failure=8
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute
Events:
  Type    Reason   Age   From             Message
  ----    ------   ----  ----             -------
  Normal  Pulled   112s  kubelet, master  Container image "k8s.gcr.io/kube-apiserver:v1.17.13" already present on machine
  Normal  Created  112s  kubelet, master  Created container kube-apiserver
  Normal  Started  111s  kubelet, master  Started container kube-apiserver
```

+ master-node: v1.18.0
+ kubelet: v1.18.0
+ api-server: v1.17.13

обновим
```
kubeadm upgrade plan

kubeadm upgrade apply v1.18.0

kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:56:30Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}

kubelet --version
Kubernetes v1.18.0

kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:58:59Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:50:46Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}

kubectl describe pod kube-apiserver-master -n kube-system
Name:                 kube-apiserver-master
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 master/10.132.0.24
Start Time:           Wed, 11 Nov 2020 18:09:47 +0000
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.132.0.24:6443
                      kubernetes.io/config.hash: df08d27978fdb766e8c7b88fbf9ccb84
                      kubernetes.io/config.mirror: df08d27978fdb766e8c7b88fbf9ccb84
                      kubernetes.io/config.seen: 2020-11-11T18:20:13.621390844Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   10.132.0.24
IPs:
  IP:           10.132.0.24
Controlled By:  Node/master
Containers:
  kube-apiserver:
    Container ID:  docker://41caaff956771622a5e7814a2aaf22e3ac1f01d16c7341c876df77aa73824888
    Image:         k8s.gcr.io/kube-apiserver:v1.18.0
    Image ID:      docker-pullable://k8s.gcr.io/kube-apiserver@sha256:fc4efb55c2a7d4e7b9a858c67e24f00e739df4ef5082500c2b60ea0903f18248
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=10.132.0.24
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
      --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
      --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
      --etcd-servers=https://127.0.0.1:2379
      --insecure-port=0
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=6443
      --service-account-key-file=/etc/kubernetes/pki/sa.pub
      --service-cluster-ip-range=10.96.0.0/12
      --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    State:          Running
      Started:      Wed, 11 Nov 2020 18:20:14 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        250m
    Liveness:     http-get https://10.132.0.24:6443/healthz delay=15s timeout=15s period=10s #success=1 #failure=8
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute
Events:
  Type    Reason   Age    From             Message
  ----    ------   ----   ----             -------
  Normal  Pulled   2m10s  kubelet, master  Container image "k8s.gcr.io/kube-apiserver:v1.18.0" already present on machine
  Normal  Created  2m10s  kubelet, master  Created container kube-apiserver
  Normal  Started  2m10s  kubelet, master  Started container kube-apiserver
```

Все гуд!

Вывод worker node из планирования
```
kubectl drain node-1 --ignore-daemonsets

kubectl get node
NAME     STATUS                     ROLES    AGE   VERSION
master   Ready                      master   41m   v1.18.0
node-1   Ready,SchedulingDisabled   <none>   22m   v1.17.4
node-2   Ready                      <none>   21m   v1.17.4
node-3   Ready                      <none>   20m   v1.17.4
```
Обновляем 
```
на выведенной воркер ноде выполняем

apt-get install -y kubelet=1.18.0-00 kubeadm=1.18.0-00

systemctl restart kubelet
```
Проверяем 
```
kubectl get node
NAME     STATUS                     ROLES    AGE   VERSION
master   Ready                      master   41m   v1.18.0
node-1   Ready,SchedulingDisabled   <none>   22m   v1.18.0
node-2   Ready                      <none>   22m   v1.17.4
node-3   Ready                      <none>   21m   v1.17.4
```
Возвращаем ноду в работу
```
kubectl uncordon node-1

kubectl get node
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   42m   v1.18.0
node-1   Ready    <none>   23m   v1.18.0
node-2   Ready    <none>   22m   v1.17.4
node-3   Ready    <none>   21m   v1.17.4
```
и т.д все остальные ноды 

Придем к этому 
```
kubectl get node
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   51m   v1.18.0
node-1   Ready    <none>   32m   v1.18.0
node-2   Ready    <none>   31m   v1.18.0
node-3   Ready    <none>   31m   v1.18.0
```
Можно удалить ресурсы 
```
gcloud compute instances delete master --zone europe-west1-b && \
gcloud compute instances delete node-1 --zone europe-west1-b && \
gcloud compute instances delete node-2 --zone europe-west1-b && \
gcloud compute instances delete node-3 --zone europe-west1-b
```

----

#### Kubespray
#### Создание нод для кластера
В GCP создайте 4 ноды с образом Ubuntu 18.04 LTS:
+ master - 1 экземпляр (n1-standard-2)
+ worker - 3 экземпляра (n1-standard-1)

Закидываем свой ssh-key (проще всего Compute Engine - Metadata - SSH Keys)

```
gcloud compute instances create master --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-2 && \
gcloud compute instances create node-1 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-1 && \
gcloud compute instances create node-2 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-1 && \
gcloud compute instances create node-3 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-1

# Для удаления
gcloud compute instances delete master --zone europe-west1-b
```

#### Клонируем репу 
```
git clone https://github.com/kubernetes-sigs/kubespray.git

# установка зависимостей
sudo pip install -r requirements.txt

# копирование примера конфига в отдельную директорию
cp -rfp inventory/sample inventory/k8s_kubespray
```
#### Правим хост файл
vi inventory.ini
```
[all]
master1 ansible_host=35.241.157.15  ip=10.132.0.28 etcd_member_name=etcd1
node1 ansible_host=104.155.89.56    # ip=10.3.0.1 etcd_member_name=etcd1
node2 ansible_host=35.195.99.29    # ip=10.3.0.1 etcd_member_name=etcd1
node3 ansible_host=34.76.161.99   # ip=10.3.0.1 etcd_member_name=etcd1

[kube-master]
master1

[etcd]
master1

[kube-node]
node1
node2
node3

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr
```
#### Применяем 
```
ansible-playbook -i inventory.ini --become --become-user=root \
    --user=lex --key-file="~/.ssh/gcp-key" cluster.yml
```
#### Пересоздаем ресурсы (Для создания 3 мастер нод + 2 воркер нод)
```
gcloud compute instances delete master --zone europe-west1-b && \
gcloud compute instances delete node-1 --zone europe-west1-b && \
gcloud compute instances delete node-2 --zone europe-west1-b && \
gcloud compute instances delete node-3 --zone europe-west1-b

gcloud compute instances create master1 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-2 && \
gcloud compute instances create master2 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-2 && \
gcloud compute instances create master3 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-2 && \
gcloud compute instances create node-1 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-1 && \
gcloud compute instances create node-2 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --zone europe-west1-b --machine-type=n1-standard-1 
```
#### Правим хост файл
vi inventory2.ini
```
[all]
master1 ansible_host=34.91.171.49  ip=10.164.0.2 etcd_member_name=etcd1
master2 ansible_host=34.91.171.49  ip=10.164.0.2 etcd_member_name=etcd1
master3 ansible_host=34.91.171.49  ip=10.164.0.2 etcd_member_name=etcd1
node1 ansible_host=35.204.111.222   # ip=10.3.0.1 etcd_member_name=etcd1
node2 ansible_host=34.90.160.12    # ip=10.3.0.1 etcd_member_name=etcd1

[kube-master]
master1
master2
master3

[etcd]
master1
master2
master3

[kube-node]
node1
node2

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr
```
#### Применяем 
```
ansible-playbook -i inventory2.ini --become --become-user=root \
    --user=lex --key-file="~/.ssh/gcp-key" cluster.yml
```
```
master1:~$ kubectl get nodes
NAME      STATUS   ROLES    AGE     VERSION
master1   Ready    master   6m30s   v1.18.2
master2   Ready    master   5m40s   v1.18.2
master3   Ready    master   5m40s   v1.18.2
node1     Ready    <none>   4m34s   v1.18.2
node2     Ready    <none>   4m34s   v1.18.2
```
#### Удаляем машины