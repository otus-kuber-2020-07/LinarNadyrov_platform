>Применяем указанный манифест + смотрим какие шаги в данный момент происходят
```
kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice
-w
```
>Проверим образ, указанный в манифест (в данном случае указанный в **deployments**)
```
kubectl get deployments.apps paymentservice -o=jsonpath='{.spec.template.spec.containers[0].image}'
```
>Проверяем образ из которого сейчас запущены pod, управляемые контроллером
```
kubectl get pods -l app=paymentservice -o=jsonpath='{.items[0:3].spec.containers[0].image}'
```
>Смотрим на историю версий нашего Deployment
```
kubectl rollout history deployment paymentservice
```
>Представим, что обновление по каким-то причинам произошло неудачно и нам необходимо сделать откат. Kubernetes предоставляет такую возможность:
```
kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l app=paymentservice -w
```
В выводе мы можем наблюдать, как происходит постепенное масштабирование вниз "нового" ReplicaSet, и масштабирование вверх "старого".

>Как автоматически отследить успешность выполнения Deployment (например для запуска в CI/CD).
В этом нам может помочь следующая команда:
```
kubectl rollout status deployment frontend
```

Команды:
```bash
kubectl cluster-info - Показывает информацию о Kubernetes master
kubectl get node     - Показывает информацию о НОДах 
kubectl get node -o wide - Показывает расширенную информацию о НОДах
kubectl get pods - Показывает информацию о pod'aх'
kubectl get pods -o wide - Показывает расширенную информацию о pod'aх' (видем на какой НОДе запущен pod)
gcloud compute ssh NAME node - Покдючаемся к нужной НОДе

kubectl describe node gke-kubia-85f6-node-0rrx - Показывает подробную информации об объекте

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