### Выполненное Д/З №2

 - [x] Основное ДЗ
 - [x] Задание со *
 - [x] Задание со **

### В процессе сделано:
- Создан и запущен локальный кластер k8s на основе [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) 
- Написан манифест ReplicaSet - ***frontend-replicaset.yaml***
- Написан Dockerfile и собран контейнер ***paymentService***
- Контейнеры помещены в [docker hub](https://hub.docker.com/repository/docker/linarnadyrov/paymentservice) 
- Написан манифест Deployment - ***paymentservice-replicaset.yaml***
- Реализован blue-green - ***paymentservice-deployment-bg.yaml***
- Реализован Reverse Rolling Update - ***paymentservice-deployment-reverse.yaml***
- Написан манифест DaemonSet для node-exporter - ***node-exporter-daemonset.yaml***

---

### Решение Д/З №2

- Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod? \
Ответ: 
Replica**Controller не умеют рестартовать запущенные поды при обновлении шаблона
- ⭐ Deployment | С использованием параметров **maxSurge** и **maxUnavailable** самостоятельно реализуйте два следующих сценария развертывания:
```
1. Аналог blue-green:
 - Развертывание трех новых pod
 - Удаление трех старых pod
```
Результат - ***paymentservice-deployment-bg.yaml***
```
2. Reverse Rolling Update:
  - Удаление одного старого pod
  - Создание одного нового pod 
  - ....
  - ....
```
Результат - ***paymentservice-deployment-reverse.yaml***
- ⭐ Написать манифест DaemonSet для node-exporter
Опробуем DaemonSet на примере [Node Exporter](https://github.com/prometheus/node_exporter)
```
1. Найдите в интернете или напишите самостоятельно манифест node-exporter-daemonset.yaml для развертывания DaemonSet с Node Exporter.
```
```
2. После применения данного DaemonSet и выполнения команды:
``` kubectl port-forward <имя любого pod в DaemonSet> 9100:9100``` метрики должны быть доступны на localhost: curl localhost:9100/metrics
```

- ⭐⭐ Дописать tolerans для DaemonSet 
``` 
Как правило, мониторинг требуется не только для worker, но и для master нод. При этом, по умолчанию, pod управляемые DaemonSet на master нодах не разворачиваются 
```
``` 
Найдите способ модернизировать свой DaemonSet таким образом, чтобы Node Exporter был развернут как на master, так и на worker нодах (конфигурацию самих нод изменять нельзя) 
```
``` 
Отразите изменения в манифесте.
```
Нужно добавить: 
```
    spec:
      tolerations:
      - operator: "Exists"
```
Результат - ***node-exporter-daemonset.yaml***


### Как проверить работоспособность:

 - Выполнить команды:
  ```
  kubectl apply -f node-exporter-daemonset.yaml 
  kubectl port-forward <имя любого pod в DaemonSet> 9100:9100
  curl localhost:9100/metrics
  ```


