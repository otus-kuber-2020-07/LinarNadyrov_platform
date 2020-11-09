#### Создание нод для кластера
В GCP создайте 4 ноды с образом Ubuntu 18.04 LTS:
+ master - 1 экземпляр (n1-standard-2)
+ worker - 3 экземпляра (n1-standard-1)

gcloud compute instances create master --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-2 

gcloud compute instances create node-1 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1

gcloud compute instances create node-2 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1 

gcloud compute instances create node-3 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1 