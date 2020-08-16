### Выполненное Д/З №2

 - [x] Основное ДЗ
 - [x] Задание со *

### В процессе сделано:
- Создан и запущен локальный кластер k8s на основе [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) 
- Написан манифест ReplicaSet - ***frontend-replicaset.yaml***
- Написан Dockerfile и собран контейнер ***paymentService***
- Контейнеры помещены в [docker hub](https://hub.docker.com/repository/docker/linarnadyrov/paymentservice) 
- Написан манифест Deployment - ***paymentservice-replicaset.yaml***
- 


---

### Решение Д/З №1

- Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod? \
Ответ: 
Replica**Controller не умеют рестартовать запущенные поды при обновлении шаблона

- Разберитесь почему все pod в namespace kube-system восстановились после удаления. Укажите причину в описании PR.Hint: core-dns и, например, kube-apiserver, имеют различия в механизме запуска и восстанавливаются по разным причинам. \
Поды кроме coredns это [static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)  управляемые напрямую kubelet'ом. Описания подов хранятся тут:
```
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
```
kubectl describe deployments.apps -n kube-system

Name:                   coredns
Namespace:              kube-system
CreationTimestamp:      Sun, 09 Aug 2020 13:28:52 +0300
Labels:                 k8s-app=kube-dns
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               k8s-app=kube-dns
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
```
- ⭐ Выясните причину, по которой pod frontend находится в статусе **Error** \
Потому, что не определено: 
```sh
panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
```


### Как проверить работоспособность:

 - Выполнить команды:
  ```shell
  kubectl port-forward --address 0.0.0.0 pod/web 8000:8000 &
  curl http://127.0.0.1:8000
  ```

 - :star: Выполнить команду:
 ```shell
 kubectl get pods -l run=frontend --field-selector=status.phase=Running
 ```

