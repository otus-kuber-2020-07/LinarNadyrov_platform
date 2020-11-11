##### Автодополнение ввода для Kubectl в BASH
```
source <(kubectl completion bash) 
# настройка автодополнения в текущую сессию bash, предварительно должен быть установлен пакет bash-completion .
echo "source <(kubectl completion bash)" >> ~/.bashrc 
# добавление автодополнения autocomplete постоянно в командную оболочку bash.
```
##### Автодополнение ввода для Kubectl в ZSH
```
source <(kubectl completion zsh)
# настройка автодополнения в текущую сессию zsh
echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)" >> ~/.zshrc 
# add autocomplete permanently to your zsh shell
```
[Шпаргалка по kubectl](https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/)

##### Основы
```bash
kubectl cluster-info         - Показывает информацию о Kubernetes master
kubectl get node             - Показывает информацию о НОДах 
kubectl get node -o wide     - Показывает расширенную информацию о НОДах
kubectl get pods             - Показывает информацию о pod'aх'
kubectl get pods -o wide     - Показывает расширенную информацию о pod'aх' (видем на какой НОДе запущен pod)
gcloud compute ssh NAME node - Покдючаемся к нужной НОДе (работает только в GCP)

kubectl describe node gke-kubia-85f6-node-0rrx - Показывает подробную информации об объекте
```
##### Формирует yaml файл через CLI 
```
create ns ns2 --dry-run -o yaml
```
##### Формирует yaml файл из запущенного Service в кластере
```
kubectl get -n monitoring service/prometheus-grafana -o yaml
```
##### Запрос получения RoleBinding для всего namespaces 
```
kubectl get rolebinding --all-namespaces
```
##### Запрос получения Role для определенного namespaces 
```
kubectl get role -n kube-system
```
##### Запрос получения ClusterRoles
```
kubectl get clusterroles
kubectl get clusterroles | grep "admin"
```
##### Создание и удаление labels 
```
kubectl label node <nodename> <labelname>=allow  - create labels for the nodes
kubectl label node <nodename> <labelname>-       - delete above labels from its respecitve nodes
```
##### За процессом можно понаблюдать с помощью 
```
kubectl get events --watch
```
##### Применяем указанный манифест + смотрим какие шаги в данный момент происходят
```
kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
```
##### Проверим образ, указанный в манифест (в данном случае указанный в **deployments**)
```
kubectl get deployments.apps paymentservice -o=jsonpath='{.spec.template.spec.containers[0].image}'
```
##### Проверяем образ из которого сейчас запущены pod, управляемые контроллером
```
kubectl get pods -l app=paymentservice -o=jsonpath='{.items[0:3].spec.containers[0].image}'
```
##### Смотрим на историю версий нашего Deployment
```
kubectl rollout history deployment paymentservice
```
##### Представим, что обновление по каким-то причинам произошло неудачно и нам необходимо сделать откат. Kubernetes предоставляет такую возможность:
```
kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l app=paymentservice -w
```
В выводе мы можем наблюдать, как происходит постепенное масштабирование вниз "нового" ReplicaSet, и масштабирование вверх "старого".

##### Как автоматически отследить успешность выполнения Deployment (например для запуска в CI/CD). В этом нам может помочь следующая команда:
```
kubectl rollout status deployment frontend

```
##### Данные из **metrics-server**
```
kubectl get pods --all-namespaces
kubectl top pods --all-namespaces
kubectl top pods --all-namespaces -l tier=control-plane --sort-by=cpu
kubectl top nodes --sort-by=cpu 
kubectl get pods --all-namespaces --show-labels
```
##### Смотрим какие есть Service
```
kubectl get service
```

##### Работа с сертификатами
```
kubectl get certificate -A 
kubectl get certificaterequests
kubectl get certificatesigningrequests
kubectl delete certificate -n harbor harbor-harbor-ingress 
```
##### Хорошая дока, правда unofficial-kubernetes
[kube-controller-manager](https://unofficial-kubernetes.readthedocs.io/en/latest/admin/kube-controller-manager/)

##### Выбор лидера в системе/управлении Kubernetes. Leader election in Kubernetes control plane
```
kubectl get endpoints - конечная точка основного ресурса
or 
kubectl get ep

kubectl get endpoints -n kube-system                      - смотрим endpoints
kubectl describe endpoints kube-scheduler -n kube-system  - смотрим leader
```
##### Удаляем все pods с нужного ns | Удаляем весь deployments с нужного ns
```
kubectl delete --all pods --namespace=foo
kubectl delete --all deployments --namespace=foo
```

##### Смотрим, что HelmRelease для микросервиса frontend появился в кластере
```
kubectl get helmrelease -n microservices-demo    - показывает успешность релиза
helm list -n microservices-demo                  - дополнительно можно таким образом посмотреть
helm history frontend -n microservices-demo      - смотрим история
```
----

##### Быстрый просмотр спецификаций ресурсов
Когда вы создаете определения ресурсов YAML, нужно знать поля и их значение для этих ресурсов.

Эти значения можно посмотреть
```
kubectl explain resource[.field]...

kubectl explain deployment
kubectl explain deployment.spec
```
##### Если вы не знаете точно, какие именно ресурсы нужны, то можете отобразить их все следующей командой 
```
kubectl api-resources
```
##### Все нижеследующие команды равнозначны
```
kubectl explain deployments.spec
# или
kubectl explain deployment.spec
# или        
kubectl explain deploy.spec
```
##### Смотрим сервисы и их endpoints 
```
kubectl get svc -A

kubectl describe svc rpc-app-service
```
##### Проброс к pod'ам 
```
ssh -L local_port:pod_ip:pod_port user@server_ip           
# пример   ssh -L 7777:10.110.10.15:3000 linarnadyrov@k1-node

kubectl -n namespaces port-forward <pod-name> <pod-port>   
# пример   kubectl -n monitoring port-forward <grafana-pod-name> 3000
```

----

### Использование кластера Kubernetes, предоставляемого как сервис с Google Kubernetes Engine.
1. Первым делом нужно ознакомиться:\
https://cloud.google.com/kubernetes-engine/docs/quickstart#cloud-shell

2. Ставим нужный софт (возможно буду дополнять)\
**ОБЯЗАТЕЛЬНО к прочтению**\
https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/#%d1%83%d1%81%d1%82%d0%b0%d0%bd%d0%be%d0%b2%d0%ba%d0%b0-kubectl-%d0%b8%d0%b7-sdk-google-cloud 

```bash
sudo apt-get install kubectl
```
3. Создаем трехузлового кластера kubernetes
```bash
gcloud container clusters create kubia --num-nodes 3 --zone us-central1-a --machine-type e2-micro
```
Указываю зону (не по умолчанию) - --zone us-central1-a

4. Запуск первого приложения на Kubernetes
```bash
kubectl run kubia --image=linarnadyrov/kubia --port=8080 --generator=run/v1
```
5. Запуск первого приложения на Kubernetes. **Второй вариант.**\
Создаем файл replicaset.yaml
```bash
kubectl create -f replicaset.yaml
```
6. Доступ к веб-приложению\
Создание объекта **Service** \
Создаем файл service.yaml
```bash
kubectl get services
```
Открываем нужные порты
```bash
gcloud compute firewall-rules create kubia --allow tcp:8080 --target-tags=k8s --description="Allow web" --direction=INGRESS
```

7. Увеличение/уменьшение количества требуемых реплик
```bash
kubectl scale replicasets.apps kubia --replicas=3
```

