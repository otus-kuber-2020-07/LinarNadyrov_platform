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
Добавим в свой профиль GitLab публичный ssh-ключ, при помощи которого flux получит доступ к нашему git-репозиторию.

Проверяем корректность работы Flux. Поместите манифест, описывающий namespace microservices-demo в
директорию deploy/namespaces и сделайте push в GitLab: 
```
│   ├── namespaces
│   │   ├── namespaces.yaml
```
Результат можно проверить через лога pod с flux и через kubectl get ns - должны увидеть новый ns. 

##### HelmRelease
Создаем и дописываем нужные параметры в этом [файле](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-gitops/kubernetes-gitops/.gitlab-ci.yml)
```
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: frontend
  namespace: microservices-demo
  annotations:
    fluxcd.io/ignore: "false"
    fluxcd.io/automated: "true"
    flux.weave.works/tag.chart-image: semver:~v0.0
spec:
  releaseName: frontend
  helmVersion: v3
  chart:
    git: git@gitlab.com:LinarNadyrov/microservices-demo.git
    ref: master
    path: deploy/charts/frontend
  values:
    image:
      repository: linarnadyrov/frontend
      tag: v0.0.1
```
Опишем некоторые части манифеста HelmRelease:
```
metadata:
...
annotations:
...
fluxcd.io/automated: "true" # 1
flux.weave.works/tag.chart-image: semver:~v0.0 # 2
spec:
releaseName: frontend
...
chart: # 3
git: git@gitlab.com:<YOUR_LOGIN>/microservices-demo.git
ref: master
path: deploy/charts/frontend
values: # 4
image:
repository: <YOUR_LOGIN>/frontend
tag: v0.0.1
```
1. Аннотация разрешает автоматическое обновление релиза в Kubernetes кластере в случае изменения версии Docker образа в Registry
2. Указываем Flux следить за обновлениями конкретных Docker образов в Registry.

Новыми считаются только образы, имеющие версию выше текущей и отвечающие маске семантического версионирования ~0.0 (например, 0.0.1, 0.0.72, но не 1.0.0)
3. Helm chart, используемый для развертывания релиза. В нашем случае указываем git-репозиторий, и директорию с чартом внутри него
4. Переопределяем переменные Helm chart. В дальнейшем Flux может сам переписывать эти значения и делать commit в git-репозиторий (например, изменять тег Docker образа при его обновлении в Registry)