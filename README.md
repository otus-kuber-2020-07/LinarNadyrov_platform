# Инфраструктурная платформа на основе Kubernetes

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

## Задание-2 
Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod? \
Ответ: 
Replica**Controller не умеют рестартовать запущенные поды при обновлении шаблона

## Доп информация 

>Применяем указанный манифест + смотрим какие шаги в данный момент происходят
```
kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice
-w
```
>Проверим образ, указанный в манифест (в данном случае указанный в **deployments**)
```
kubectl get deployments.apps paymentservice -o=jsonpath='{.spec.template.spec.containers[0].image}'
```
>Проверяем образ из которого сейчас запущены pod, управляемые контроллером
```
kubectl get pods -l app=paymentservice -o=jsonpath='{.items[0:3].spec.containers[0].image}'
```
>Смотрим на историю версий нашего Deployment
```
kubectl rollout history deployment paymentservice
```
>Представим, что обновление по каким-то причинам произошло неудачно и нам необходимо сделать откат. Kubernetes предоставляет такую возможность:
```
kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l app=paymentservice -w
```
В выводе мы можем наблюдать, как происходит постепенное масштабирование вниз "нового" ReplicaSet, и масштабирование вверх "старого".

>Как автоматически отследить успешность выполнения Deployment (например для запуска в CI/CD).
В этом нам может помочь следующая команда:
```
kubectl rollout status deployment frontend
```

## Deployment | Задание-2 со ⭐

С использованием параметров **maxSurge** и **maxUnavailable** самостоятельно реализуйте два следующих сценария развертывания:
1. **Аналог blue-green:**
 - Развертывание трех новых pod
 - Удаление трех старых pod
2. **Reverse Rolling Update:**
  - Удаление одного старого pod
  - Создание одного нового pod 
  - ....
  - ....

## DaemonSet | Задание-2 со ⭐

Опробуем DaemonSet на примере [Node Exporter](https://github.com/prometheus/node_exporter)
1. Найдите в интернете или напишите самостоятельно манифест node-exporter-daemonset.yaml для развертывания DaemonSet с Node Exporter.
2. После применения данного DaemonSet и выполнения команды:
``` kubectl port-forward <имя любого pod в DaemonSet> 9100:9100``` метрики должны быть доступны на localhost: curl localhost:9100/metrics

## DaemonSet | Задание-2 со ⭐⭐

- Как правило, мониторинг требуется не только для worker, но и для master нод. При этом, по умолчанию, pod управляемые DaemonSet на master нодах не разворачиваются
- Найдите способ модернизировать свой DaemonSet таким образом, чтобы Node Exporter был развернут как на master, так и на worker нодах (конфигурацию самих нод изменять нельзя)
- Отразите изменения в манифесте. 

Нужно добавить: 
```
    spec:
      tolerations:
      - operator: "Exists"
```