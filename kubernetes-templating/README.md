### Выполненное Д/З №7

 - [x] Основное ДЗ
 - [x] Одно задание со *

ДЗ выполняется в GCP. Создаем free аккаунт и настраиваем подключение. 

Первым делом нужно [ознакомиться](https://cloud.google.com/kubernetes-engine/docs/quickstart#cloud-shell)

Настроить [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/#%d1%83%d1%81%d1%82%d0%b0%d0%bd%d0%be%d0%b2%d0%ba%d0%b0-kubectl-%d0%b8%d0%b7-sdk-google-cloud), если это ранее не сделано. 

Команды в [GCP](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)

Создаем k8s: 
```
gcloud container clusters create homework-7 --num-nodes 1 --zone us-central1-a --machine-type e2-medium
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
### Устанавливаем готовые Helm charts. Будем работать со следующими сервисами:
- [nginx-ingress](https://github.com/helm/charts/tree/master/stable/nginx-ingress) сервис, обеспечивающий доступ к публичным ресурсам кластера (переехал [сюда](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/))
- [cert-manager](https://github.com/jetstack/cert-manager/tree/master/deploy/charts/cert-manager) сервис, позволяющий динамически генерировать Let's Encrypt сертификаты для ingress ресурсов

Обязательно к чтению данную [документацию](https://cert-manager.io/docs/installation/kubernetes/). Как нужно настраивать [ACME](https://cert-manager.io/docs/configuration/acme/)
- [chartmuseum](https://github.com/helm/charts/tree/master/stable/chartmuseum) специализированный репозиторий для хранения helm charts
- [harbor](https://github.com/goharbor/harbor-helm) хранилище артефактов общего назначения (Docker Registry),поддерживающее helm charts

Устанавливаем helm. После устанавливаем готовые Helm charts.
- nginx-ingress 

Создадим namespace и release nginx-ingress
```
kubectl create ns nginx-ingress
helm upgrade --install nginx-ingress stable/nginx-ingress --wait --namespace=nginx-ingress -version=1.41.3
```
### cert-manager
Добавим репозиторий: 
```
helm repo add jetstack https://charts.jetstack.io
```
Установим cert-manager:
```
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.16.0 \
  --set installCRDs=true
```
Для корректной работы реализован манифест - [letsencrypt-production.yaml](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-templating/kubernetes-templating/cert-manager/letsencrypt-production.yaml)

### chartmuseum
Кастомизируем установку chartmuseum


Реализуем [values.yaml](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-templating/kubernetes-templating/chartmuseum/values.yaml) на основе [ориганального файла](https://github.com/helm/charts/blob/master/stable/chartmuseum/values.yaml)

Файл values.yaml включает в себя:
- Создание ingress ресурса с корректным hosts.name (должен использоваться nginx-ingress)
- Автоматическую генерацию Let's Encrypt сертификата

Установим chartmuseum:
```
kubectl create ns chartmuseum
helm upgrade --install chartmuseum stable/chartmuseum --wait \
--namespace=chartmuseum \
--version=2.13.2 \
-f kubernetes-templating/chartmuseum/values.yaml

Проверим, что release chartmuseum установился:
helm ls -n chartmuseum
```
Критерий успешности установки: 
- Chartmuseum доступен по URL https://chartmuseum.DOMAIN
- Сертификат для данного URL валиден

### Установите harbor в кластер с использованием helm3
Для этого: 
- Реализуем файл [values.yaml](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-templating/kubernetes-templating/harbor/values.yaml)
- 
```
helm repo add harbor https://helm.goharbor.io
helm repo update
kubectl create ns harbor
helm upgrade --install harbor harbor/harbor --wait \
--namespace=harbor \
--version=1.1.2 \
-f kubernetes-templating/harbor/values.yaml
```
Реквизиты по умолчанию: admin/Harbor12345

