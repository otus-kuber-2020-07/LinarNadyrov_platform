### В ходе работы мы:
- Напишем CustomResource и CustomResourceDefinition для mysql оператора
- 🐍 Напишем часть логики mysql оператора при помощи python KOPF
- Сделаем соберем образ и сделаем деплой оператора.

##### Подготовка 
```
git clone git@github.com:LinarNadyrov/small_k8s.git 
cd small_k8s/vagrant/two
vagrant up
vagrant ssh k1s-master -c 'cat /home/vagrant/.kube/config' > ~/.kube/config
```
Все манифесты будут лежать в директории kubernetes-operators/deploy

Cоздадим CustomResource deploy/cr.yml со следующим содержимым:
```
apiVersion: otus.homework/v1
kind: MySQL
metadata:
  name: mysql-instance
spec:
  image: mysql:5.7
  database: otus-database
  password: otuspassword  # Так делать не нужно, следует использовать secret
  storage_size: 1Gi
usless_data: "useless info"
```
Применим и видим, что не работает. 
```
error: unable to recognize "deploy/cr.yml": no matches for kind "MySQL" in version
"otus.homework/v1"

Ошибка связана с отсутсвием объектов типа MySQL в API kubernetes.
```
Создадим CRD deploy/crd.yml 

`CustomResourceDefinition` - это `ресурс` для определения других `ресурсов`

```
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
name: mysqls.otus.homework # имя CRD должно иметь формат plural.group
spec:
scope: Namespaced # Данный CRD будер работать в рамках namespace
group: otus.homework # Группа, отражается в поле apiVersion CR
versions: # Список версий
- name: v1
served: true # Будет ли обслуживаться API-сервером данная версия
storage: true # Версия описания, которая будет сохраняться в etcd
names: # различные форматы имени объекта CR
kind: MySQL # kind CR
plural: mysqls
singular: mysql
shortNames:
- ms
```

Применим объекты. 

C созданными объектами можно взаимодействовать через kubectl:
```
kubectl get crd
kubectl get mysqls.otus.homework
kubectl describe mysqls.otus.homework mysql-instance
```
На данный момент мы никак не описали схему нашего `CustomResource`. Объекты типа mysql могут иметь абсолютно произвольные поля, нам бы хотелось этого избежать, для этого будем использовать `validation`. Для начала удалим CR mysql-instance 
```
kubectl delete mysqls.otus.homework mysql-instance
```
Добавим в спецификацию CRD ( `spec` ) параметры `validation`. Конечный вариант [тут](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-operators/kubernetes-operators/deploy/crd.yml)

Пробуем применить CRD и CR
```
kubectl apply -f deploy/crd.yml
kubectl apply -f deploy/cr.yml

#
error: error validating "deploy/cr.yml": error validating data:
ValidationError(MySQL): unknown field "usless_data" in homework.otus.v1.MySQL; if
you choose to ignore these errors, turn validation off with --validate=false
```
Убираем из cr.yml и применяем: 
```
usless_data: "useless info"

kubectl apply -f deploy/cr.yml 
# Ошибок больше нет
```
### Задание по CRD:
Если сейчас из описания mysql убрать строчку из спецификации, то манифест будет принят API сервером. Для того, чтобы этого избежать, добавьте описание обязательный полей в `CustomResourceDefinition`. 
> Подсказка. Пример есть в лекции.

Конечный вариант [тут](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-operators/kubernetes-operators/deploy/crd.yml). Смотрим ( `spec` )

Применяем.

### Операторы 
- Оператор включает в себя `CustomResourceDefinition` и `сustom сontroller`
  + CRD содержит описание объектов CR
  + Контроллер следит за объектами определенного типа, и осуществляет всю логику работы оператора 
- CRD мы уже создали далее будем писать свой контроллер (все задания по написанию контроллера дополнительными)
- Используем готовый контроллер

##### Описание контроллера
Используемый нами контроллер будет обрабатывать два типа событий:
- При создании объекта типа ( `kind: mySQL` ), он будет:
```
* Cоздавать PersistentVolume, PersistentVolumeClaim, Deployment, Service для mysql
* Создавать PersistentVolume, PersistentVolumeClaim для бэкапов базы данных, если их еще нет.
* Пытаться восстановиться из бэкапа
```
- При удалении объекта типа ( `kind: mySQL` ), он будет:
```
* Удалять все успешно завершенные backup-job и restore-job
* Удалять PersistentVolume, PersistentVolumeClaim, Deployment, Service для mysql
```
