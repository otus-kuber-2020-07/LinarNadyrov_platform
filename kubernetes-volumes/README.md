### Выполненное Д/З №6

- [x] Основное ДЗ
- [x] Одно задание со *

ДЗ выполняется на основе kind

Запуск 
```
kind create cluster
export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
```
#### В этом ДЗ мы развернем StatefulSet c [MinIO](https://min.io/) - локальным S3 хранилищем
minio-statefulset.yaml
В результате применения конфигурации должно произойти следующее:
- Запуститься под с MinIO
- Создаться PVC
- Динамически создаться PV на этом PVC с помощью дефолотного StorageClass

Для того, чтобы наш StatefulSet был доступен изнутри кластера, создадим Headless Service - minio-headlessservice.yaml 
```
kubectl get statefulsets
kubectl get pods
kubectl get pvc
kubectl get pv
kubectl describe <resource> <resource_name>
```

#### ⭐ В конфигурации нашего StatefulSet данные указаны в открытом виде, что не безопасно. Поместите данные в secrets и настройте конфигурацию на их использование.
- файл secret.yaml
- плюс исравленный файл minio-statefulset.yaml