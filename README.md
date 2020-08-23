## Инфраструктурная платформа на основе Kubernetes
[Полезные команды](cli/README.md)

### 1. Kubernetes-intro
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

### 2. Kubernetes-controllers
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

### 3. Kubernetes-security
### Задание:
- Создание ServiceAccount
- Создание Namespaces
- Создание Role, RoleBinding
- Создание ClusterRole, ClusterRoleBinding
#### Полезные ссылки 
- [Описание выполненного ДЗ](kubernetes-security/README.md)
- [Понимаем RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/422801/)
- [RBAC в Kubernetes](https://rtfm.co.ua/kubernetes-znakomstvo-chast-5-rbac-avtorizaciya-i-primery-role-i-rolebinding/#Kubernetes_RBAC_%E2%80%94_%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80)

### 4. Kubernetes-networks
### Задание: 
- Добавляем проверок Pod с помощью readinessProbe и livenessProbe

#### Полезные ссылки 
- [Описание выполненного ДЗ](kubernetes-networks/README.md)
- [Настройка Liveness, Readiness и Startup проб](https://kubernetes.io/ru/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Kubernetes Services and Iptables](https://msazure.club/kubernetes-services-and-iptables/)
- [Bare-metal considerations](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)
- 
