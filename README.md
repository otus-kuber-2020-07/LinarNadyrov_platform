## Задание-1

Разберитесь почему все pod в namespace kube-system восстановились после удаления. Укажите причину в описании PR.Hint: core-dns и, например, kube-apiserver, имеют различия в механизме запуска и восстанавливаются по разным причинам

## Решение:
Поды кроме coredns это [static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)  управляемые напрямую kubelet'ом. Описания подов хранятся тут:
```bash
$ ls -lah /etc/kubernetes/manifests/
total 16K
drwxr-xr-x 2 root root  120 Aug  9 10:28 .
drwxr-xr-x 4 root root  160 Aug  9 10:28 ..
-rw------- 1 root root 1.9K Aug  9 10:28 etcd.yaml
-rw------- 1 root root 3.0K Aug  9 10:28 kube-apiserver.yaml
-rw------- 1 root root 2.5K Aug  9 10:28 kube-controller-manager.yaml
-rw------- 1 root root 1.1K Aug  9 10:28 kube-scheduler.yaml
```

pod coredns описан в деплойменте: 
```bash 
kubectl describe deployments.apps -n kube-system

Name:                   coredns
Namespace:              kube-system
CreationTimestamp:      Sun, 09 Aug 2020 13:28:52 +0300
Labels:                 k8s-app=kube-dns
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               k8s-app=kube-dns
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
```
## Задание-1 со ⭐
Выясните причину, по которой pod frontend находится в статусе **Error** \
Потому, что не определено: 
```sh
panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
```
