### Выполненное Д/З №2

 - [x] Основное ДЗ
 - [x] Задание со *

### В процессе сделано:
- Создан и запущен локальный кластер k8s на основе [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) 
- Написан манифест ReplicaSet - ***frontend-replicaset.yaml***
- Написан Dockerfile и собран контейнер ***paymentService***
- Контейнеры помещены в [docker hub](https://hub.docker.com/repository/docker/linarnadyrov/paymentservice) 
- Написан манифест Deployment - ***paymentservice-replicaset.yaml***
- 


---

### Решение Д/З №1

- Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod? \
Ответ: 
Replica**Controller не умеют рестартовать запущенные поды при обновлении шаблона
- 







### Как проверить работоспособность:

 - Выполнить команды:
  ```shell
  kubectl port-forward --address 0.0.0.0 pod/web 8000:8000 &
  curl http://127.0.0.1:8000
  ```

 - :star: Выполнить команду:
 ```shell
 kubectl get pods -l run=frontend --field-selector=status.phase=Running
 ```

