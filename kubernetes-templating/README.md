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
helm upgrade --install nginx-ingress stable/nginx-ingress --wait --namespace=nginx-ingress --version=1.41.3
```
### cert-manager
Добавим репозиторий: 
```
helm repo add jetstack https://charts.jetstack.io
```
Установим cert-manager:
```
kubectl create ns cert-manager
helm upgrade --install \
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
- Установил harbor:
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

Критерий успешности установки: 
- Chartmuseum доступен по URL https://harbor.DOMAIN
- Сертификат для данного URL валиден

Обратите внимание, как helm3 хранит информацию о release:
```
kubectl get secrets -n harbor -l owner=helm
```
### Создаем свой helm chart
#### Типичная жизненная ситуация:
- У вас есть приложение, которое готово к запуску в Kubernetes
- У вас есть манифесты для этого приложения, но вам надо запускать его на разных окружениях с разными параметрами
#### Возможные варианты решения:
- Написать разные манифесты для разных окружений
- Использовать "костыли" - sed, envsubst, etc...
- Использовать полноценное решение для шаблонизации (helm, etc...)

#### Мы рассмотрим третий вариант. Возьмем готовые манифесты и подготовим их к релизу на разные окружения.
Использовать будем демо-приложение [hipster-shop](https://github.com/GoogleCloudPlatform/microservices-demo), представляющее собой типичный набор микросервисов.

Стандартными средствами helm инициализируйте структуру директории с содержимым будущего helm chart
```
helm create kubernetes-templating/hipster-shop
```
Мы будем создавать chart для приложения с нуля, поэтому удалите values.yaml и содержимое templates. После этого перенесите файл all-hipster-shop.yaml в директорию templates.

В целом, helm chart уже готов, вы можете попробовать установить его:
```
kubectl create ns hipster-shop
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
```

Сейчас наш helm chart hipster-shop совсем не похож на настоящий. При этом, все микросервисы устанавливаются из одного файла all-hipstershop.yaml

Давайте исправим это и первым делом займемся микросервисом frontend. Скорее всего он разрабатывается отдельной командой, а исходный код хранится в отдельном репозитории.

Создадим заготовку:
```
helm create kubernetes-templating/frontend
```
Аналогично чарту hipster-shop удалите файл values.yaml и файлы в директории templates, создаваемые по умолчанию. 
Выделим из файла all-hipster-shop.yaml манифесты для установки микросервиса frontend. В директории templates чарта frontend создайте файлы: 
- deployment.yaml, service.yaml, ingress.yaml

После того, как вынесете описание deployment и service для frontend из файла all-hipster-shop.yaml переустановите chart hipster-shop и проверьте, что доступ к UI пропал и таких ресурсов больше нет.

Установите chart frontend в namespace hipster-shop и проверьте что доступ к UI вновь появился:
```
helm upgrade --install frontend kubernetes-templating/frontend --namespace hipster-shop
```

#### Пришло время минимально шаблонизировать наш chart frontend
- выносим данные в переменную (смотри файл values.yaml)

Теперь наш frontend стал немного похож на настоящий helm chart. Не стоит забывать, что он все еще является частью одного большого микросервисного приложения hipster-shop. Поэтому было бы неплохо включить его в зависимости этого
приложения.

Для начала, удалите release frontend из кластера:
```
helm list -a -A
helm delete frontend -n hipster-shop
```

В Helm 3 список зависимостей рекомендуют объявлять в файле Chart.yaml

Добавьте chart frontend как зависимость
```
dependencies:
  - name: frontend
    version: 0.1.0
    repository: "file://../frontend" - ссылается на /frontend
```
Обновим зависимости:
```
helm dep update kubernetes-templating/hipster-shop
```
В директории kubernetes-templating/hipster-shop/charts появился архив frontend-0.1.0.tgz содержащий chart frontend определенной версии и добавленный в chart hipster-shop как зависимость. Обновите release hipster-shop и убедитесь, что ресурсы frontend вновь созданы.
```
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
```
## Таким образом мы можем микросервисное приложение выносить в отдельную разработку и добавлять при необходмости

Осталось понять, как из CI-системы мы можем менять параметры helm chart, описанные в values.yaml. Для этого существует специальный ключ --set. Изменим NodePort для frontend в release, не меняя его в самом chart:
```
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace
hipster-shop --set frontend.service.NodePort=31234
```

⭐ Выберите сервисы, которые можно установить как зависимости, используя community chart's. Например, это может быть Redis. Реализуйте их установку через Chart.yaml и обеспечьте сохранение работоспособности приложения.
- Убираем redis из hister-shop/templates/all-hipster-shop.yaml
- Создаем ```helm create redis```
- Добавляем зависимости hister-shop/Chart.yaml и обновляем эти зависимости ```helm dep update kubernetes-templating/hipster-shop```
- В директории kubernetes-templating/hipster-shop/charts должен появится архив redis-****
- Обновляем release hipster-shop ```helm upgrade --install hipster-shop hipster-shop --namespace hipster-shop```
