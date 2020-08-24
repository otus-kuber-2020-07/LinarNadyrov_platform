### Выполненное Д/З №4 

Данное задание выполняется в minikube , версии не ниже 1.0.0. \
Для MacOS рекомендуется использовать драйвер [VM HyperKit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/).

Выполнение ДЗ | План работы
Работа с тестовым веб-приложением
- Добавление проверок Pod
- Создание объекта Deployment
- Добавление сервисов в кластер ( ClusterIP )
- Включение режима балансировки IPVS

Доступ к приложению извне кластера
- Установка MetalLB в Layer2-режиме
- Добавление сервиса LoadBalancer
- Установка Ingress-контроллера и прокси ingress-nginx
- Создание правил Ingress

#### Добавление проверок Pod
Добавляем readinessProbe и livenessProbe. Получаем файл web-pod.yml
- Создание объекта Deployment 
#### Создали файл web-deploy.yaml, сделали шаблон конфигурации пода, исправили readinessProbe = 8000 и replicas = 3, добавили RollingUpdate.
Узнал, что можно наблюдать за процессом с помощью 
```
kubectl get events --watch
```
#### Добавление сервисов в кластер ( ClusterIP )
- Создал файл web-svc-cip.yaml 

#### Установка MetalLB
MetalLB позволяет запустить внутри кластера L4-балансировщик, который будет принимать извне запросы к сервисам и раскидывать их между подами. Установка его проста:
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
kubectl create secret generic -n metallb-system memberlist --fromliteral=secretkey="$(openssl rand -base64 128)"
```
Проверяем, что были созданы нужные объекты:
```
kubectl --namespace metallb-system get all
```
Настройка балансировщика с помощью ConfigMap

В конфигурации настраиваем:
- Режим L2 (анонс адресов балансировщиков с помощью ARP)
- Создаем пул адресов 172.17.255.1 - 172.17.255.255 - они будут назначаться сервисам с типом LoadBalancer \

Смотри файл - metallb-config.yaml, web-svc-lb.yaml 

#### ⭐ DNS через MetalLB
- Сделайте сервис LoadBalancer , который откроет доступ к CoreDNS снаружи кластера (позволит получать записи через внешний IP).

Например, nslookup web.default.cluster.local 172.17.255.10.
- Поскольку DNS работает по TCP и UDP протоколам - учтите это в конфигурации. Оба протокола должны работать по одному и тому же IP-адресу балансировщика.
- Полученные манифесты положите в подкаталог ./coredns

#### Создание Ingress
- Установка начинается с основного манифеста:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingressnginx/master/deploy/static/provider/baremetal/deploy.yaml
```
- После установки основных компонентов, в рекомендуется применить манифест, который создаст NodePort -сервис. Но у нас есть MetalLB, мы можем сделать круче.

Получаем манифест - nginx-lb.yaml 

#### Подключение приложение Web к Ingress
Получаем манифест - web-svc-headless.yaml

#### Создание правил Ingress
Получаем манифест - web-ingress.yaml 

#### ⭐ Ingress для Dashboard - НЕ ВЫПОЛНЕНО 
- Добавьте доступ к kubernetes-dashboard через наш Ingress-прокси:
- Cервис должен быть доступен через префикс /dashboard ).
- Kubernetes Dashboard должен быть развернут из официального манифеста. Актуальная ссылка есть в репозитории проекта
- Написанные вами манифесты положите в подкаталог ./dashboard

#### ⭐ | Canary для Ingress - НЕ ВЫПОЛНЕНО 
Реализуйте канареечное развертывание с помощью ingress-nginx:
- Перенаправление части трафика на выделенную группу подов должно происходить по HTTP-заголовку.
Документация [тут](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md#canary)
- Естественно, что вам понадобятся 1-2 "канареечных" пода.
- Написанные манифесты положите в подкаталог ./canary