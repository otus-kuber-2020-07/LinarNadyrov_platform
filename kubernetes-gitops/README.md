Подготовка Kubernetes кластера | Задание со ⭐
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
```
