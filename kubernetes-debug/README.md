#### Подготовка 
Запускаем локально наш мини кластер k8s 
```
git clone git@github.com:LinarNadyrov/small_k8s.git 
cd small_k8s/vagrant/two
vagrant up
vagrant ssh k1s-master -c 'cat /home/vagrant/.kube/config' > ~/.kube/config
```

#### kubectl debug | Задание
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
        - containerPort: 10027
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
bash-5.0# зы
bash: зы: command not found
bash-5.0# ps
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   29 101       0:00 nginx: worker process
   44 root      0:00 bash
   51 root      0:00 ps
bash-5.0#
```