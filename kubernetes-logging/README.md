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
Добавляем pool в существующий k8s: 
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
В данном домашнем задании мы будет устанавливать и использовать различные решения для логирования различными способами.Начнем с "классического" набора инструментов (ElasticSearch,Fluent Bit, Kibana) и "классического" способа его установки в Kubernetes кластер (Helm).
Рекомендуемый репозиторий с Helm chart для ElasticSearch и Kibana на текущий момент - https://github.com/elastic/helm-charts

Добавим его:
```
helm repo add elastic https://helm.elastic.co
```
Запустим каждую реплику ElasticSearch на своей, выделенной ноде из infra-pool. Создадим файл [elasticsearch.values.yaml]
