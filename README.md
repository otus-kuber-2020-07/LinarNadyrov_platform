## Инфраструктурная платформа на основе Kubernetes
### 1. CLI утилита kubectl для управления kubernetes.
[Полезные команды](cli/README.md)

### 2. Kubernetes-intro
### Задание: 
- Разберитесь почему все pod в namespace kube-system восстановились после удаления. Укажите причину в описании PR.Hint: core-dns и, например, kube-apiserver, имеют различия в механизме запуска и восстанавливаются по разным причинам. 
- ⭐ Выясните причину, по которой pod frontend находится в статусе **Error**

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

#### Полезные ссылки
- [Описание выполненного ДЗ](kubernetes-templating/README.md)
- Команды в [GCP](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)
- [Красивая настройка VScode](https://raw.githubusercontent.com/Jasstkn/.dotfiles/master/.zshrc)
- [ChartMuseum](https://github.com/helm/chartmuseum), репозиторий в [github](https://github.com/helm/charts/tree/master/stable/chartmuseum)
- 