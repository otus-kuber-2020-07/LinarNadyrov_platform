##### Подготовка Kubernetes кластера | Задание со ⭐
- Автоматизируйте создание Kubernetes кластера
- Кластер должен разворачиваться после запуска pipeline в GitLab
- Инфраструктурный код и файл .gitlab-ci.yaml поместите в отдельный репозиторий и приложите ссылку на данный репозиторий в PR

Для автоматизации было сделано: 
- создана, настроена виртуальная машина и подключена как runner в gitlab. 

Сам файл [.gitlab-ci.yaml](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-gitops/kubernetes-gitops/.gitlab-ci.yml)
```
before_script:
  - df -h
  - docker info
  - docker-compose --version
 
stages:
  - create
  
createJob:
  stage: create
  tags:
    - k8s
  script:
    - gcloud beta container clusters create k8s-gitops --zone europe-west1-b --num-nodes=4 --machine-type=n1-standard-2 --disk-size=50GB --addons=Istio --istio-config=auth=MTLS_PERMISSIVE
    - kubectl get node
    - gcloud container clusters list
    - kubectl get service -n istio-system
```
Команда для удаления 
```
gcloud container clusters delete k8s-gitops --zone europe-west1-b
```

##### GitOps. Установка Flux
Установим CRD, добавляющую в кластер новый ресурс - HelmRelease:
```
kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/master/deploy/crds.yaml
```
Добавим официальный репозиторий Flux
```
helm repo add fluxcd https://charts.fluxcd.io
```
Произведем установку Flux в кластер, в namespace flux
```
kubectl create namespace flux
helm upgrade --install flux fluxcd/flux -f flux.values.yaml --namespace flux
```
Установим Helm operator
```
helm upgrade --install helm-operator fluxcd/helm-operator -f helm-operator.values.yaml --namespace flux
```
Получаем значение ключа следующей командой (необходимо установить fluxctl):
```
fluxctl identity --k8s-fwd-ns flux
```
