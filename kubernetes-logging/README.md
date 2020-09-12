#### Выполненное Д/З №10

 - [x] Самостоятельное задание | Установка nginx-ingress
 - []

#### Для выполнения домашнего задания понадобится подготовка Kubernetes кластера:
Мы планируем отдать три из четырех нод кластера под инфраструктурные сервисы. Присвоим этим нодам определенный [Taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/), чтобы избежать запуска на них случайных pod. Укажем следующую конфигурацию taint через web-интерфейс GCP: node-role=infra:NoSchedule (я буду назначать все через CLI).

```
gcloud compute zones list 
```

Создаем k8s: 
```
gcloud container clusters create logging-hw --num-nodes 1 \
    --zone us-west1-a --machine-type n1-standard-2 \
    --disk-size=50GB --no-enable-stackdriver-kubernetes
``` 
Добавляем pool и node-taints в существующий k8s: 
```
gcloud container node-pools create infra-pool --cluster logging-hw --zone us-west1-a --num-nodes 3 --machine-type n1-standard-2 --disk-size=50GB --node-taints node-role=infra:NoSchedule
```

Удаляем k8s
```
gcloud container clusters delete logging-hw --zone us-west1-a
```
В результате должна получиться следующая конфигурация кластера:
```
kubectl get node
NAME                                        STATUS   ROLES    AGE     VERSION
gke-logging-hw-default-pool-753d6383-qxrq   Ready    <none>   2m49s   v1.15.12-gke.2
gke-logging-hw-infra-pool-48573e6a-468g     Ready    <none>   16s     v1.15.12-gke.2
gke-logging-hw-infra-pool-48573e6a-f3kw     Ready    <none>   57s     v1.15.12-gke.2
gke-logging-hw-infra-pool-48573e6a-ggzw     Ready    <none>   57s     v1.15.12-gke.2
```
#### Установка HipsterShop
Установим в Kubernetes кластер уже знакомый HipsterShop. Самый простой способ сделать это - применить подготовленный манифест: 
```
kubectl create ns microservices-demo
kubectl apply -f https://raw.githubusercontent.com/express42/otus-platformsnippets/master/Module-02/Logging/microservices-demo-without-resources.yaml -n microservices-demo

Проверим, что все pod развернулись на ноде из default-pool:
kubectl get pods -n microservices-demo -o wide
```

#### Установка EFK стека | Helm charts
В данном домашнем задании мы будет устанавливать и использовать различные решения для логирования различными способами. Начнем с "классического" набора инструментов (ElasticSearch,Fluent Bit, Kibana) и "классического" способа его установки в Kubernetes кластер (Helm).
Рекомендуемый репозиторий с Helm chart для ElasticSearch и Kibana на текущий момент - https://github.com/elastic/helm-charts

Добавим его:
```
helm repo add elastic https://helm.elastic.co
```
Запустим каждую реплику ElasticSearch на своей, выделенной ноде из infra-pool. Создадим файл [elasticsearch.values.yaml](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-logging/kubernetes-logging/elasticsearch.values.yaml), будем указывать в этом файле нужные нам values.


Обратимся к файлу values.yaml в [репозитории](https://github.com/elastic/helm-charts/tree/master/elasticsearch) и найдем там ключ tolerations. Мы помним, что ноды из infra-pool имеют taint node-role=infra:NoSchedule. Давайте разрешим
ElasticSearch запускаться на данных нодах. Для этого внесем данные в файл [elasticsearch.values.yaml](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-logging/kubernetes-logging/elasticsearch.values.yaml)

```
tolerations:
  - key: node-role
    operator: Equal
    value: infra
    effect: NoSchedule
```
Установим elasticsearch:
```
kubectl create ns observability
helm upgrade --install elasticsearch elastic/elasticsearch --namespace observability -f elasticsearch.values.yaml
```

Теперь ElasticSearch может запускаться на нодах из infra-pool, но это не означает, что он должен это делать.
Исправим этот момент и добавим в [elasticsearch.values.yaml](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-logging/kubernetes-logging/elasticsearch.values.yaml) NodeSelector, определяющий, на каких нодах мы можем запускать наши pod.
```
nodeSelector:
cloud.google.com/gke-nodepool: infra-pool
```
Обновим установленный elasticsearch:
```
helm upgrade --install elasticsearch elastic/elasticsearch --namespace observability -f elasticsearch.values.yaml
```
Другой, и, на самом деле, более гибкий способ осуществить задуманное - [nodeAffinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

Должный получить следующую картину:
```
kubectl get pods -n observability -o wide -l chart=elasticsearch 
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE                                      NOMINATED NODE   READINESS GATES
elasticsearch-master-0   1/1     Running   0          56m   10.44.3.2   gke-logging-hw-infra-pool-48573e6a-468g   <none>           <none>
elasticsearch-master-1   1/1     Running   0          57m   10.44.2.2   gke-logging-hw-infra-pool-48573e6a-ggzw   <none>           <none>
elasticsearch-master-2   1/1     Running   0          59m   10.44.1.2   gke-logging-hw-infra-pool-48573e6a-f3kw   <none>           <none>
```
#### Установка nginx-ingress | Самостоятельное задание
Для того, чтобы продолжить установку EFK стека и получить доступ к Kibana, предварительно потребуется развернуть ingress-controller.

Установите nginx-ingress. Должно быть развернуто три реплики controller, по одной, на каждую ноду из infra-pool

```
kubectl create ns nginx-ingress
helm upgrade --install nginx-ingress stable/nginx-ingress --namespace=nginx-ingress --version=1.41.3 -f nginx-ingress.values.yaml
```
После установки получил 
*******************************************************************************************************
* DEPRECATED, please use https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx *
*******************************************************************************************************

