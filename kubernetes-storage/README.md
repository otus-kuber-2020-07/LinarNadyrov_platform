#### Выполненное Д/З №22
 - [x]
       * Создать StorageClass для CSI Host Path Driver
       * Создать объект PVC c именем `storage-pvc`
       * Создать объект Pod c именем `storage-pod`
       * Хранилище нужно смонтировать в `/data`
 
Разворачиваем мини кластер куба: 
```
git clone git@github.com:LinarNadyrov/small_k8s.git 
cd small_k8s/vagrant/two
vagrant up
vagrant ssh k1s-master -c 'cat /home/vagrant/.kube/config' > ~/.kube/config
```

Ставим HostPath Driver:

Официльная [документация](https://github.com/kubernetes-csi/csi-driver-host-path/blob/master/docs/deploy-1.17-and-later.md)

Задеплоим CDR для поддержки snapshot в кластер, согласно официальной документации. 

```
# Apply VolumeSnapshot CRDs
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v2.0.1/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v2.0.1/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v2.0.1/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml

# Create snapshot controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v2.0.1/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v2.0.1/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
```

Задеплоим CSI HostPath Driver:
```
git clone git@github.com:kubernetes-csi/csi-driver-host-path.git
cd csi-driver-host-path
bash deploy/kubernetes-latest/deploy-hostpath.sh
```

Смотрим
```
kubectl get pods -A
NAME                         READY   STATUS    RESTARTS   AGE
csi-hostpath-attacher-0      1/1     Running   0          4m24s
csi-hostpath-provisioner-0   1/1     Running   0          4m21s
csi-hostpath-resizer-0       1/1     Running   0          4m20s
csi-hostpath-snapshotter-0   1/1     Running   0          4m19s
csi-hostpath-socat-0         1/1     Running   0          4m18s
csi-hostpathplugin-0         3/3     Running   0          4m22s
snapshot-controller-0        1/1     Running   0          18m
```

Запустим пример:
```
for i in ./examples/csi-storageclass.yaml ./examples/csi-pvc.yaml ./examples/csi-app.yaml; do kubectl apply -f $i; done
```
Проверяем: 
```
kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS      REASON   AGE
pvc-6946a04b-0dd5-4810-b03a-4d562a9d119b   1Gi        RWO            Delete           Bound    default/csi-pvc       csi-hostpath-sc            5h8m

kubectl get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
csi-pvc       Bound    pvc-6946a04b-0dd5-4810-b03a-4d562a9d119b   1Gi        RWO            csi-hostpath-sc   5h8m
```

```
kubectl describe pods/my-csi-app
Name:         my-csi-app
Namespace:    default
Priority:     0
Node:         k1s-master/192.168.33.100
Start Time:   Sat, 07 Nov 2020 15:13:36 +0300
Labels:       <none>
Annotations:  cni.projectcalico.org/podIP: 10.244.111.14/32
              cni.projectcalico.org/podIPs: 10.244.111.14/32
Status:       Running
IP:           10.244.111.14
IPs:
  IP:  10.244.111.14
Containers:
  my-frontend:
    Container ID:  docker://709d972b780770013150abbadb6d90f5b60e183323106de019c518ab6dd24eb8
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:a9286defaba7b3a519d585ba0e37d0b2cbee74ebfe590960b0b1d6a5e97d1e1d
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      1000000
    State:          Running
      Started:      Sat, 07 Nov 2020 15:13:46 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from my-csi-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-vtpj6 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  my-csi-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  csi-pvc
    ReadOnly:   false
  default-token-vtpj6:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-vtpj6
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                  Age        From                     Message
  ----    ------                  ----       ----                     -------
  Normal  Scheduled               <unknown>                           Successfully assigned default/my-csi-app to k1s-master
  Normal  SuccessfulAttachVolume  51s        attachdetach-controller  AttachVolume.Attach succeeded for volume "pvc-6946a04b-0dd5-4810-b03a-4d562a9d119b"
```
Самые интересные секции:
- `Containers.my-frontend.Mounts` - в контейнер замонтирован volume my-csi-volume в директорию /data.
- `Volunes` - my-csi-volume это персистентное хранилице мозданое по заявке csi-pvc
- `Events` - в ивентах можно увидеть, что volume успешно приатачен.

Проверим как работает HostPath driver:

Создадим файл в /data директории внутри конейнера в поде my-csi-app:

```
kubectl exec -it my-csi-app -- /bin/sh

touch /data/test1.txt
```
Проверяем, что файлик появился в нашем HostPath контейнере:
```
vagrant ssh k1s-master

vagrant@k1s-master:~$ sudo find / -name test1.txt
/var/lib/kubelet/pods/15aa4210-35d4-4b64-90e0-ad6e7d453134/volumes/kubernetes.io~csi/pvc-6946a04b-0dd5-4810-b03a-4d562a9d119b/mount/test1.txt
/var/lib/csi-hostpath-data/a95a1b20-20f2-11eb-b8ec-7e87f59cf654/test1.txt
```
Все отлично. 

Переходим к ДЗ. 

Задание:
* Создать StorageClass для CSI Host Path Driver
* Создать объект PVC c именем `storage-pvc`
* Создать объект Pod c именем `storage-pod`
* Хранилище нужно смонтировать в `/data`

```
cd /kubernetes-storage
for i in ./hw/01storageClass.yaml ./hw/02pvc.yaml ./hw/03pod.yaml; do kubectl apply -f $i; done
```
Проверяем
```
kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS      REASON   AGE
pvc-6946a04b-0dd5-4810-b03a-4d562a9d119b   1Gi        RWO            Delete           Bound    default/csi-pvc       csi-hostpath-sc            5h22m
pvc-7bbaad7f-f53b-486f-b5bb-0e7fd6a7a1fa   1Gi        RWO            Delete           Bound    default/storage-pvc   otus-homework22            45m

kubectl get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
csi-pvc       Bound    pvc-6946a04b-0dd5-4810-b03a-4d562a9d119b   1Gi        RWO            csi-hostpath-sc   5h23m
storage-pvc   Bound    pvc-7bbaad7f-f53b-486f-b5bb-0e7fd6a7a1fa   1Gi        RWO            otus-homework22   46m
```

```
kubectl describe pods/storage-pod
Name:         storage-pod
Namespace:    default
Priority:     0
Node:         k1s-master/192.168.33.100
Start Time:   Sat, 07 Nov 2020 19:50:55 +0300
Labels:       <none>
Annotations:  cni.projectcalico.org/podIP: 10.244.111.15/32
              cni.projectcalico.org/podIPs: 10.244.111.15/32
Status:       Running
IP:           10.244.111.15
IPs:
  IP:  10.244.111.15
Containers:
  app:
    Container ID:  docker://5e096b23b952aa5246c755a2f3cb6b7b5de8b6deae386b21361e1dd36868d6f9
    Image:         bash
    Image ID:      docker-pullable://bash@sha256:01fad26fa8ba21bce6e8c47222acfdb54649957f1e86d53a0c8e03360271abf6
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      10000000
    State:          Running
      Started:      Sat, 07 Nov 2020 19:51:15 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from data-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-vtpj6 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  data-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  storage-pvc
    ReadOnly:   false
  default-token-vtpj6:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-vtpj6
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                  Age        From                     Message
  ----     ------                  ----       ----                     -------
  Warning  FailedScheduling        <unknown>                           0/2 nodes are available: 2 pod has unbound immediate PersistentVolumeClaims.
  Warning  FailedScheduling        <unknown>                           0/2 nodes are available: 2 pod has unbound immediate PersistentVolumeClaims.
  Normal   Scheduled               <unknown>                           Successfully assigned default/storage-pod to k1s-master
  Normal   SuccessfulAttachVolume  38s        attachdetach-controller  AttachVolume.Attach succeeded for volume "pvc-7bbaad7f-f53b-486f-b5bb-0e7fd6a7a1fa"
  Normal   Pulling                 29s        kubelet, k1s-master      Pulling image "bash"
  Normal   Pulled                  18s        kubelet, k1s-master      Successfully pulled image "bash" in 10.599620147s
  Normal   Created                 18s        kubelet, k1s-master      Created container app
  Normal   Started                 18s        kubelet, k1s-master      Started container app
```
Отлично. Работает. 

Протестируем функционал снапшотов:
```
kubectl apply -f 04VolumeSnapshotClass.yaml
```
Делаем снапшот:
```
kubectl apply -f 05snapshotVolume.yaml
```
Проверяем:
```
kubectl describe volumesnapshot
Name:         test-snapshot
Namespace:    default
Labels:       <none>
Annotations:  API Version:  snapshot.storage.k8s.io/v1beta1
Kind:         VolumeSnapshot
Metadata:
  Creation Timestamp:  2020-11-07T17:03:25Z
  Finalizers:
    snapshot.storage.kubernetes.io/volumesnapshot-as-source-protection
    snapshot.storage.kubernetes.io/volumesnapshot-bound-protection
  Generation:  1
  Managed Fields:
    API Version:  snapshot.storage.k8s.io/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:source:
          .:
          f:persistentVolumeClaimName:
        f:volumeSnapshotClassName:
    Manager:      kubectl
    Operation:    Update
    Time:         2020-11-07T17:03:25Z
    API Version:  snapshot.storage.k8s.io/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"snapshot.storage.kubernetes.io/volumesnapshot-as-source-protection":
          v:"snapshot.storage.kubernetes.io/volumesnapshot-bound-protection":
      f:status:
        .:
        f:boundVolumeSnapshotContentName:
        f:creationTime:
        f:readyToUse:
        f:restoreSize:
    Manager:         snapshot-controller
    Operation:       Update
    Time:            2020-11-07T17:03:26Z
  Resource Version:  60528
  Self Link:         /apis/snapshot.storage.k8s.io/v1beta1/namespaces/default/volumesnapshots/test-snapshot
  UID:               2333ce84-3b55-4e26-b37c-c8b9317ebf4c
Spec:
  Source:
    Persistent Volume Claim Name:  storage-pvc
  Volume Snapshot Class Name:      test-snapclass
Status:
  Bound Volume Snapshot Content Name:  snapcontent-2333ce84-3b55-4e26-b37c-c8b9317ebf4c
  Creation Time:                       2020-11-07T17:03:26Z
  Ready To Use:                        true
  Restore Size:                        1Gi
Events:
  Type     Reason             Age    From                 Message
  ----     ------             ----   ----                 -------
  Warning  ErrorPVCFinalizer  7m37s  snapshot-controller  Error check and remove PVC Finalizer for VolumeSnapshot
```
Видим ошибку. Копать не стал.  
