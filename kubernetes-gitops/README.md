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
Результат можно проверить через лог pod с flux и через kubectl get ns - должны увидеть новый ns. 

##### HelmRelease
Создаем и дописываем нужные параметры в этом [файле](https://gitlab.com/LinarNadyrov/microservices-demo/-/blob/master/deploy/releases/frontend.yaml)
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

3. Helm chart, используемый для развертывания релиза. В нашем случае указываем git-репозиторий, и директорию с чартом внутри него.
4. Переопределяем переменные Helm chart. В дальнейшем Flux может сам переписывать эти значения и делать commit в git-репозиторий (например, изменять тег Docker образа при его обновлении в Registry)

##### Смотрим, что HelmRelease для микросервиса frontend появился в кластере
```
kubectl get helmrelease -n microservices-demo    - показывает успешность релиза
helm list -n microservices-demo                  - дополнительно можно таким образом посмотреть
helm history frontend -n microservices-demo      - смотрим история
```
Можно инициировать синхронизацию вручную
```
fluxctl --k8s-fwd-ns flux sync
```
Обновление образа
- Внесите изменения в исходный код микросервиса frontend (не имеет значения, какие) и пересоберите образ, при этом инкрементировав версию тега (до v0.0.2)
- Дождитесь автоматического обновления релиза в Kubernetes кластере (для просмотра ревизий релиза можно использовать команду helm
history frontend -n microservices-demo )
- Проверьте, изменилось ли что-либо в git-репозитории (в частности, в файле deploy/releases/frontend.yaml )
```
helm history frontend -n microservices-demo
REVISION	UPDATED                 	STATUS    	CHART          	APP VERSION	DESCRIPTION     
1       	Mon Oct 12 10:10:57 2020	superseded	frontend-0.21.0	1.16.0     	Install complete
2       	Mon Oct 12 10:21:21 2020	deployed  	frontend-0.21.0	1.16.0     	Upgrade complete
```
для PR
```
ts=2020-10-12T10:53:17.592863632Z caller=release.go:79 component=release release=frontend targetNamespace=microservices-demo resource=microservices-demo:helmrelease/frontend helmVersion=v3 info="starting sync run"
ts=2020-10-12T10:53:17.957844168Z caller=release.go:353 component=release release=frontend targetNamespace=microservices-demo resource=microservices-demo:helmrelease/frontend helmVersion=v3 info="running upgrade" action=upgrade
ts=2020-10-12T10:53:18.001977035Z caller=helm.go:69 component=helm version=v3 info="preparing upgrade for frontend" targetNamespace=microservices-demo release=frontend
ts=2020-10-12T10:53:18.007413208Z caller=helm.go:69 component=helm version=v3 info="resetting values to the chart's original version" targetNamespace=microservices-demo release=frontend
ts=2020-10-12T10:53:18.269994608Z caller=helm.go:69 component=helm version=v3 info="performing update for frontend" targetNamespace=microservices-demo release=frontend
ts=2020-10-12T10:53:18.335937104Z caller=helm.go:69 component=helm version=v3 info="creating upgraded release for frontend" targetNamespace=microservices-demo release=frontend
ts=2020-10-12T10:53:18.34588903Z caller=helm.go:69 component=helm version=v3 info="checking 1 resources for changes" targetNamespace=microservices-demo release=frontend
ts=2020-10-12T10:53:18.354244147Z caller=helm.go:69 component=helm version=v3 info="Looks like there are no changes for Deployment \"frontend\"" targetNamespace=microservices-demo release=frontend
ts=2020-10-12T10:53:18.363187657Z caller=helm.go:69 component=helm version=v3 info="updating status for upgraded release for frontend" targetNamespace=microservices-demo release=frontend
ts=2020-10-12T10:53:18.396955717Z caller=release.go:364 component=release release=frontend targetNamespace=microservices-demo resource=microservices-demo:helmrelease/frontend helmVersion=v3 info="upgrade succeeded" revision=883b3033c919ffafbf11362981f307379d7e8d37 phase=upgrade
```

##### Canary deployments с Flagger и Istio
Установка Flagger
```
helm repo add flagger https://flagger.app
kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artifacts/flagger/crd.yaml

helm upgrade --install flagger flagger/flagger \
--namespace=istio-system \
--set crd.create=false \
--set meshProvider=istio \
--set metricsServer=http://prometheus:9090
```
Измените созданное ранее описание namespace microservices-demo (добавляем labels)
```
apiVersion: v1
kind: Namespace
metadata:
  name: microservices-demo
  labels:
    istio-injection: enabled
```
После синхронизации проверку можно выполнить командой
```
kubectl get ns microservices-demo --show-labels
```
Самый простой способ добавить sidecar контейнер в уже запущенные pod - удалить их
```
kubectl delete pods --all -n microservices-demo
```
Проверка 
```
kubectl get pods -n microservices-demo
NAME                                READY   STATUS    RESTARTS   AGE
frontend-primary-6cfbc69b94-79pk8   2/2     Running   0          35m
```
Добавляем Istio Gateway и VirtualService [frontend-gw.yaml/frontend-vs.yaml](https://gitlab.com/LinarNadyrov/microservices-demo/-/tree/master/deploy/charts/frontend/templates) для сервиса frontend.

Добавляем ресурс canary [canary.yaml](https://gitlab.com/LinarNadyrov/microservices-demo/-/tree/master/deploy/charts/frontend/templates) для сервиса frontend.

Проверим, что Flagger:

Успешно инициализировал canary ресурс frontend: 
```
kubectl get canary -n microservices-demo
NAME       STATUS        WEIGHT   LASTTRANSITIONTIME
frontend   Initialized   0        2020-10-18T13:45:34Z
```
Обновил pod, добавив ему к названию постфикс primary:
```
kubectl get pods -n microservices-demo -l app=frontend-primary
NAME                                READY   STATUS    RESTARTS   AGE
frontend-primary-6cfbc69b94-79pk8   2/2     Running   0          59m
```
Выполняем canary deploy сервиса frontend.
```
Events:
  Type    Reason  Age   From     Message
  ----    ------  ----  ----     -------
  Normal  Synced  60m   flagger  Initialization done! frontend.microservices-demo

kubectl get canaries -n microservices-demo frontend
NAME       STATUS      WEIGHT   LASTTRANSITIONTIME
frontend   Succeeded   0        2020-10-18T14:15:17Z
```
