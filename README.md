## Инфраструктурная платформа на основе Kubernetes
### 1. CLI утилита kubectl для управления kubernetes.
[Полезные команды](cli/README.md)

### 2. Kubernetes-intro
### Задание: 
- Разберитесь почему все pod в namespace kube-system восстановились после удаления. Укажите причину в описании PR.Hint: core-dns и, например, kube-apiserver, имеют различия в механизме запуска и восстанавливаются по разным причинам. 
-   Выясните причину, по которой pod frontend находится в статусе **Error**

#### Полезные ссылки

- [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/) 
- [minikube](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/)
- [StaticPods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
- [Pods Overview](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)
- [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- [Описание выполненного ДЗ](kubernetes-intro/README.md)

### 3. Kubernetes-controllers
### Задание:
- Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod?
- ⭐ Deployment | С использованием параметров **maxSurge** и **maxUnavailable** самостоятельно реализуйте два следующих сценария развертывания:
   #### Аналог blue-green:
   Развертывание трех новых pod \
   Удаление трех старых pod

   #### Reverse Rolling Update:
   Удаление одного старого pod \
   Создание одного нового pod \
   ....

- ⭐ Написать манифест DaemonSet для node-exporter
- ⭐⭐ Дописать tolerans для DaemonSet

#### Полезные ссылки 
- [kind](https://kind.sigs.k8s.io/)
- [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Deployment Update Strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy)
- [Blue-green strategy](https://www.redhat.com/en/topics/devops/what-is-blue-green-deployment)
- [ReadinessProbe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes)
- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [Node-exporter](https://github.com/prometheus/node_exporter)
- [Taints and Toleration](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#concepts)
- [Описание выполненного ДЗ](kubernetes-controllers/README.md)

### 4. Kubernetes-security
### Задание:
- Создание ServiceAccount
- Создание Namespaces
- Создание Role, RoleBinding
- Создание ClusterRole, ClusterRoleBinding
#### Полезные ссылки 
- [Описание выполненного ДЗ](kubernetes-security/README.md)
- [Понимаем RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/422801/)
- [RBAC в Kubernetes](https://rtfm.co.ua/kubernetes-znakomstvo-chast-5-rbac-avtorizaciya-i-primery-role-i-rolebinding/#Kubernetes_RBAC_%E2%80%94_%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80)

### 5. Kubernetes-networks
### Задание: 
- Добавление проверок для Pod с помощью readinessProbe и livenessProbe
- Создание объекта Deployment с шаблоном конфигурации Pod, исправление readinessProbe = 8000 и replicas = 3, добавление RollingUpdate.
- Создание объекта Service. Включение IPVS. 
- Установка MetalLB. 
- DNS через MetalLB. 
- Создание Ingress
- Ingress для Dashboard
- Canary для Ingress

#### Полезные ссылки 
- [Описание выполненного ДЗ](kubernetes-networks/README.md)
- [Настройка Liveness, Readiness и Startup проб](https://kubernetes.io/ru/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Kubernetes Services and Iptables](https://msazure.club/kubernetes-services-and-iptables/)
- [Bare-metal considerations](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)
- [Canary для Ingress](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md#canary)

### 6. Kubernetes-volume
### Задание: 
- Запуск kind. Разварачивание StatefulSet c [minIO](https://min.io/).
- Работа с Secrets. 

#### Полезные ссылки
- [Описание выполненного ДЗ](kubernetes-volumes/README.md)
- [StatefulSet Basics](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Создание сервиса для открытия доступа к приложению](https://kubernetes.io/ru/docs/tutorials/kubernetes-basics/expose/expose-intro/)
- [Connecting Applications with Services](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/)
- [Метки и селекторы](https://kubernetes.io/ru/docs/concepts/overview/working-with-objects/labels/)
- [Учебные примеры](https://github.com/shamshev/otus/tree/master/k8s-2020-07/volumes)

### 7. Kubernetes-templating
### Задание: 
- Создаем кластер k8s в GCP. Настраиваем рабочее окружение. 
- Создаем сервис nginx-ingress
- Создаем сервис cert-manager
- Создаем сервис chartmuseum
- Создаем сервис harbor
- Создаем свой helm chart
- ⭐Создаем свой helm chart которое можно установить как зависимость

#### Полезные ссылки
- [Описание выполненного ДЗ](kubernetes-templating/README.md)
- Команды в [GCP](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)
- [Красивая настройка VScode](https://raw.githubusercontent.com/Jasstkn/.dotfiles/master/.zshrc)
- [ChartMuseum](https://github.com/helm/chartmuseum), репозиторий в [github](https://github.com/helm/charts/tree/master/stable/chartmuseum)
- [Helm list](https://helm.sh/docs/helm/helm_list/)

### 8. Kubernetes-operators
### Задание:


### 9. Kubernetes-monitoring
### Задание:
- Cоздать кастомный образ nginx с nginx-exporter и запускаем его.
- Установить и настроить [prometheus-operator](https://github.com/prometheus-operator/prometheus-operator#:~:text=Overview,monitoring%20stack%20for%20Kubernetes%20clusters.).

#### Полезные ссылки
- [Описание выполненного ДЗ](kubernetes-monitoring/README.md)
- [kube-prometheus](https://hub.helm.sh/charts/choerodon/kube-prometheus) (или по другому **prometheus-operator**)
- Что такое [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/getting-started.md)

### 10. Kubernetes-logging
### Задание:
- Установить EFK stack
- Создать dashboard в kibana
- Мониторинг ElasticSearch
- Визуализация в kibana (вкладка Visualize с типом TSVB)  

#### Полезные ссылки
- [Описание выполненного ДЗ](kubernetes-logging/README.md)
- [Создание node-pools в GCP](https://cloud.google.com/sdk/gcloud/reference/container/node-pools/create)
- [Добавление node-pools в GCP](https://cloud.google.com/kubernetes-engine/docs/how-to/node-pools#gcloud_2)
- [Информация про Taint](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#taint)
- [nodeAffinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
- [kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx)
- [Elasticsearch Helm Chart](https://github.com/elastic/helm-charts/tree/master/elasticsearch)
- [fluent-bit github](https://github.com/helm/charts/blob/master/stable/fluent-bit/values.yaml) и [fluent-bit official](https://fluentbit.io/documentation/0.13/output/elasticsearch.html)
- [PDB](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)
- [Больше метрик в Elasticsearch](https://habr.com/ru/company/yamoney/blog/358550/)
- [Log format Nginx](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/log-format/) и [тут](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#log-format-escape-json)

### 15. Kubernetes-GitOps
### Задание:
- В качестве хранилища кода и CI-системы в этом домашнем задании мы будем использовать SaaS [GitLab](https://gitlab.com/LinarNadyrov/microservices-demo)
- Автоматизируем создание Kubernetes кластера с помощью GitLab
- GitOps с помощью инструмента Flux
- Canary deployments с Flagger и Istio (мне было тяжеловато)

#### Полезные ссылки
- [Описание выполненного ДЗ](kubernetes-gitops/README.md)
- [fluxcd](https://github.com/fluxcd/helm-operator-get-started)
- [Официальная документация Flux](https://docs.fluxcd.io/en/1.21.0/)
- [Canary Custom Resource](https://docs.flagger.app/usage/how-it-works#canary-custom-resource)
- [Terraform Provider для Helm](https://github.com/hashicorp/terraform-provider-helm)
- [Argo CD](https://argoproj.github.io/)
- [Документация Flux](https://docs.fluxcd.io/en/stable/)
- [Документация Helm-Operator](https://docs.fluxcd.io/projects/helm-operator/en/latest/index.html)
- [Документация Flagger](https://docs.flagger.app/tutorials/istio-progressive-delivery)
- [Документация Istio](https://istio.io/latest/docs/)

### 18. Kubernetes-vault
### Задание:
- Установим кластер vault в kubernetes
- Научимся создавать секреты и политики
- Настроим авторизацию в vault через kubernetes sa
- Сделаем под с контейнером nginx, в который прокинем секреты из vault через consul-template

#### Полезные ссылки
- [Описание выполненного ДЗ](kubernetes-vault/README.md)
- [vault](https://learn.hashicorp.com/tutorials/vault/agent-kubernetes#step-1-create-a-service-account)
- [vault-guides](https://github.com/hashicorp/vault-guides)

### 22. Kubernetes-storage
### Задание:
- Установить CSI-драйвер и протестировать функционал снапшотов
- 🌟развернуть k8s-кластер, к которому добавить хранилище на iSCSI

#### Полезные ссылки
- [Описание выполненного ДЗ](kubernetes-storage/README.md)

