## Инфраструктурная платформа на основе Kubernetes

### 1. Kubernetes-intro
### Задание: 
- Разберитесь почему все pod в namespace kube-system восстановились после удаления. Укажите причину в описании PR.Hint: core-dns и, например, kube-apiserver, имеют различия в механизме запуска и восстанавливаются по разным причинам. 
- ⭐ Выясните причину, по которой pod frontend находится в статусе **Error**

#### Полезные ссылки

- [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/) 
- [minikube](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/)
- [StaticPods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
- [Pods Overview](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)
- [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- [Описание выполненного ДЗ](kubernetes-intro/README.md)

### 2. Kubernetes-controllers
### Задание:
- Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod?
- ⭐ Deployment | С использованием параметров maxSurge и maxUnavailable реализоватьуйте два следующих сценария развертывания: \ 
   **Аналог blue-green:**
 - Развертывание трех новых pod
 - Удаление трех старых pod
   **Reverse Rolling Update:**
  - Удаление одного старого pod
  - Создание одного нового pod 
  - ....
  - ....
- 


#### Полезные ссылки 
- [kind](https://kind.sigs.k8s.io/)
- [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Deployment Update Strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy)
- [Blue-green strategy](https://www.redhat.com/en/topics/devops/what-is-blue-green-deployment)
- [ReadinessProbe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes)
- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [Node-exporter](https://github.com/prometheus/node_exporter)
- [Taints and Toleration](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#concepts)
- [Описание выполненного ДЗ](kubernetes-controllers/README.md)

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