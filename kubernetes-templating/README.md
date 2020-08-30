### Выполненное Д/З №7

 - [x] Основное ДЗ
 - [x] Одно задание со *

ДЗ выполняется в GCP. Создаем free аккаунт и настраиваем подключение. 

Первым делом нужно [ознакомиться](https://cloud.google.com/kubernetes-engine/docs/quickstart#cloud-shell)

Настроить [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/#%d1%83%d1%81%d1%82%d0%b0%d0%bd%d0%be%d0%b2%d0%ba%d0%b0-kubectl-%d0%b8%d0%b7-sdk-google-cloud), если это ранее не сделано. 

Команды в [GCP](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)

Создаем k8s: 
```
gcloud container clusters create homework-7 --num-nodes 1 --zone us-central1-a --machine-type e2-micro
```
Написано, создай кластер k8s с именем c одной worker НОДой в таком регионе с конфигурацией машин из e2-micro (самые дешевые машины, так как используют ЦП на основе доступности)

После чего смотрим: 
```
kubectl get node -o wide
NAME                                  STATUS   ROLES    AGE     VERSION          INTERNAL-IP   EXTERNAL-IP    OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-otus-default-pool-ed20e05d-h5k3   Ready    <none>   6m47s   v1.15.12-gke.2   10.128.0.3    34.122.37.74   Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-otus-default-pool-ed20e05d-ljp4   Ready    <none>   6m47s   v1.15.12-gke.2   10.128.0.4    34.123.228.5   Container-Optimized OS from Google   4.19.112+        docker://19.3.1 
```
Удаляем k8s:
```
gcloud container clusters delete homework-7 --zone us-central1-a
```

