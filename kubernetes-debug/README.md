#### Подготовка 
Запускаем локально наш мини кластер k8s 
```
git clone git@github.com:LinarNadyrov/small_k8s.git 
cd small_k8s/vagrant/two
vagrant up
vagrant ssh k1s-master -c 'cat /home/vagrant/.kube/config' > ~/.kube/config
```

### kubectl debug | Задание
- Установите в ваш кластер kubectl debug:
    + Выполнив ```brew install aylei/tap/kubectl-debug```
    + Или скачав архив [отсюда](https://github.com/aylei/kubectl-debug/releases) с исполняемым файлом и добавив его в $PATH
- Запустите в кластере поды с агентом kubectl-debug из [этого](https://raw.githubusercontent.com/aylei/kubectl-debug/dd7e4965e4ae5c4f53e6cf9fd17acc964274ca5c/scripts/agent_daemonset.yml) манифеста
- Проверьте работу команды strace на любом поде (можно использовать Web-сервер из предыдущих заданий)
- Определите, почему не работает strace и почините
- Необходимые для выполнения задания файлы положите в папку kubernetes-debug/strace

Локально установил ```brew install aylei/tap/kubectl-debug```

Запускаем pod c Web-сервер
```
kubectl run nginx --image=nginx --port=80 -n default
```

Пробывал запускать поды с агентом kubectl-debug с разных манифестов: 
```
манифест который завелся - agent_daemonset.yaml 

# Обратить внимание 
-  containerPort: 10027
   hostPort: 32767
```
И при запуске debug ругался 
```
kubectl debug -n default nginx --port-forward
Agent Pod info: [Name:debug-agent-pod-a6282eca-2693-11eb-a74f-acde48001122, Namespace:default, Image:aylei/debug-agent:latest, HostPort:10027, ContainerPort:10027]
Waiting for pod debug-agent-pod-a6282eca-2693-11eb-a74f-acde48001122 to run...
Forwarding from 127.0.0.1:10027 -> 10027
Forwarding from [::1]:10027 -> 10027
Handling connection for 10027
                             Start deleting agent pod nginx
error execute remote, Internal error occurred: error attaching to container: Error: No such image: docker.io/nicolaka/netshoot:latest
error: Internal error occurred: error attaching to container: Error: No such image: docker.io/nicolaka/netshoot:latest
```
Проблема ушла после того как на хосте сделал
```
sudo docker pull docker.io/nicolaka/netshoot:latest 
```
Версия мини кластера 
```
kubectl get node
NAME         STATUS   ROLES    AGE   VERSION
k1s-master   Ready    master   13d   v1.19.3
k1s-node     Ready    <none>   13d   v1.19.3
```

#### Запускаем debug
```
kubectl debug -n default nginx --port-forward

# Проверка
kubectl debug -n default nginx --port-forward
Agent Pod info: [Name:debug-agent-pod-387a0256-2696-11eb-ac99-acde48001122, Namespace:default, Image:aylei/debug-agent:latest, HostPort:10027, ContainerPort:10027]
Waiting for pod debug-agent-pod-387a0256-2696-11eb-ac99-acde48001122 to run...
Forwarding from 127.0.0.1:10027 -> 10027
Forwarding from [::1]:10027 -> 10027
Handling connection for 10027
                             container created, open tty...
bash-5.0# ps
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   29 101       0:00 nginx: worker process
   55 root      0:00 bash
   80 root      0:00 ps
bash-5.0#
bash-5.0#
bash-5.0# strace -c -p1
strace: Process 1 attached
^Cstrace: Process 1 detached

bash-5.0#
bash-5.0# strace -c -p29
strace: Process 29 attached
```

### iptables-tailer 
Он предназначен для того, чтобы выводить информацию об отброшенных iptables пакетах в журнал событий Kubernetes ( kubectl get events )

Основной кейс - сообщить разработчикам сервисов о проблемах с NetworkPolicy.

#### Чтобы выполнить домашнее задание нам потребуется следующее:
- Кластер k8s c запущенным Calico
- Тестовое приложение
- Инсталляция kube-iptables-tailer
- Результаты работы должны быть в папке kubernetes-debug/kit

#### iptables-tailer | Тестовое приложение
- Для нашего задания в качестве тестового приложения вы возьмем [netperf-operator](https://github.com/piontec/netperf-operator)
    + Это Kubernetes-оператор, который позволяет запускать тесты пропускной способности сети между нодами кластера
    + Сам проект - не очень production-grade, но иногда выручает netperf-operator
- Установите манифесты для запуска оператора в кластере (лежат в папке deploy в репозитории проекта):
    + Custom Resource Definition - схема манифестов для запуска тестов Netperf
    + RBAC - политики и разрешения для нашего оператора
    + И сам оператор, который будет следить за появлением ресурсов с Kind: Netperf и запускать поды с клиентом и сервером утилиты NetPerf

#### Установка [netperf-operator](https://github.com/piontec/netperf-operator)
```
git clone https://github.com/piontec/netperf-operator.git 
```
#### Применяем манифесты
```
kubectl apply -f ./deploy/crd.yaml
kubectl apply -f ./deploy/rbac.yaml
kubectl apply -f ./deploy/operator.yaml
```
#### Запускаем пример
```
kubectl apply -f ./deploy/cr.yaml

# Смотрим результат 
kubectl describe netperf.app.example.com/example
# Нам нужно Done
Status:
  Client Pod:          netperf-client-874b00ff94df
  Server Pod:          netperf-server-874b00ff94df
  Speed Bits Per Sec:  12662.5
  Status:              Done
```
#### Применяем политику (включаем логирование в iptables) и смотрим как меняется вывод
```
kubectl apply -f kit/netperf-calico-policy.yaml
kubectl delete -f ./deploy/cr.yaml
kubectl apply -f ./deploy/cr.yaml

# Смотрим результат 
kubectl describe netperf.app.example.com/example
# Нам нужно (обратить внимание - Started test, не Done)
Status:
  Client Pod:          netperf-client-2469f954d876
  Server Pod:          netperf-server-2469f954d876
  Speed Bits Per Sec:  0
  Status:              Started test
```
#### Подключаемся к НОДе по SSH
```
sudo iptables --list -nv | grep DROP - счетчики дропов 
sudo iptables --list -nv | grep LOG
sudo journalctl -k | grep calico
```
#### Запустим iptables-tailer
```
kubectl apply -f kit/iptables-tailer.yaml 
# Смотрим результат
kubectl describe daemonset kube-iptables-tailer -n kube-system

Events:
  Type     Reason        Age                 From                  Message
  ----     ------        ----                ----                  -------
  Warning  FailedCreate  10s (x13 over 31s)  daemonset-controller  Error creating: pods "kube-iptables-tailer-" is forbidden: error looking up service account kube-system/kube-iptables-tailer: serviceaccount "kube-iptables-tailer" not found
```
#### Применим ServiceAccount
```
kubectl apply -f kit/kit-serviceaccount.yaml
kubectl apply -f kit/kit-clusterrole.yaml
kubectl apply -f kit/kit-clusterrolebinding.yaml

# Чуток подождем и смотрим
kubectl describe daemonset kube-iptables-tailer -n kube-system
```
#### Пересоздадим netperf
```
kubectl delete -f ./deploy/cr.yaml
kubectl apply -f ./deploy/cr.yaml
```
