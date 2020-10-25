#### Создаем k8s (минимум 3 НОДы): 
```
gcloud container clusters create k8s-vault --num-nodes 3 \
    --zone europe-west1-b --machine-type n1-standard-2 \
    --disk-size=50GB
```

Команда для удаления k8s (на всякий случай)
```
gcloud container clusters delete k8s-vault --zone europe-west1-b
```

#### Инсталляция hashicorp vault HA в k8s
```
git clone https://github.com/hashicorp/consul-helm.git
helm upgrade --install consul-helm consul-helm/
```

```
git clone https://github.com/hashicorp/vault-helm.git
```
Отредактируем параметры установки в values.yaml
```
standalone:
enabled: false
....
ha:
enabled: true
...
ui:
enabled: true
serviceType: "ClusterIP"
```

```
helm upgrade --install vault vault-helm -f vault-helm/values.yaml
```

Вывод helm status vault 
```
helm status vault
NAME: vault
LAST DEPLOYED: Sat Oct 24 17:54:00 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault
```
Статус pods: 
```
kubectl get pod -l app.kubernetes.io/instance=vault
NAME                                    READY   STATUS    RESTARTS   AGE
vault-0                                 0/1     Running   0          3m2s
vault-1                                 0/1     Running   0          3m2s
vault-2                                 0/1     Running   0          3m2s
vault-agent-injector-56bf46695f-sj854   1/1     Running   0          3m2s
```
Инициализируем vault: 
```
kubectl exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1
Unseal Key 1: dqixkNoNlf93kiN6I1ojr958aSYA3u2qHhyiXBJCKYY=

Initial Root Token: s.ddEdoJW5upyMKT4RHib5SoAo

Vault initialized with 1 key shares and a key threshold of 1. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 1 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 1 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

Статус параметров Initialized, Sealed:
```
kubectl exec -it vault-0 -- vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       1
Threshold          1
Unseal Progress    0/1
Unseal Nonce       n/a
Version            1.5.4
HA Enabled         true
```

UNSEAL
```
kubectl exec -it vault-0 -- vault operator unseal
Unseal Key (will be hidden):
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.5.4
Cluster Name    vault-cluster-2281a528
Cluster ID      ff81e604-b6ce-bc11-ad0b-b5bd25880ce4
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active

➜  ~ kubectl exec -it vault-1 -- vault operator unseal
Unseal Key (will be hidden):
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.5.4
Cluster Name           vault-cluster-2281a528
Cluster ID             ff81e604-b6ce-bc11-ad0b-b5bd25880ce4
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.44.2.6:8200

➜  ~ kubectl exec -it vault-2 -- vault operator unseal
Unseal Key (will be hidden):
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.5.4
Cluster Name           vault-cluster-2281a528
Cluster ID             ff81e604-b6ce-bc11-ad0b-b5bd25880ce4
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.44.2.6:8200
```

STATUS: 
```
kubectl exec -it vault-0 -- vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.5.4
Cluster Name    vault-cluster-2281a528
Cluster ID      ff81e604-b6ce-bc11-ad0b-b5bd25880ce4
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active

➜  ~ kubectl exec -it vault-1 -- vault status
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.5.4
Cluster Name           vault-cluster-2281a528
Cluster ID             ff81e604-b6ce-bc11-ad0b-b5bd25880ce4
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.44.2.6:8200

➜  ~ kubectl exec -it vault-2 -- vault status
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.5.4
Cluster Name           vault-cluster-2281a528
Cluster ID             ff81e604-b6ce-bc11-ad0b-b5bd25880ce4
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.44.2.6:8200
```

Залогинимся в vault (у нас есть root token)
```
kubectl exec -it vault-0 -- vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.ddEdoJW5upyMKT4RHib5SoAo
token_accessor       iVY3dabYQ9zJKGDQ1tBGlio6
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
➜  ~
➜  ~ kubectl exec -it vault-0 -- vault auth list
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_14d7d5fd    token based credentials
```

Заведем секреты:
```
kubectl exec -it vault-0 -- vault secrets enable --path=otus kv
kubectl exec -it vault-0 -- vault secrets list --detailed
kubectl exec -it vault-0 -- vault kv put otus/otus-ro/config username='otus' password='asajkjkahs'
kubectl exec -it vault-0 -- vault kv put otus/otus-rw/config username='otus' password='asajkjkahs'
kubectl exec -it vault-0 -- vault read otus/otus-ro/config
kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config
```

```
kubectl exec -it vault-0 -- vault read otus/otus-ro/config
Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus

➜  ~ kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config
====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```

Включим авторизацию черерз k8s: 
```
kubectl exec -it vault-0 -- vault auth enable kubernetes
Success! Enabled kubernetes auth method at: kubernetes/

➜  ~ kubectl exec -it vault-0 -- vault auth list
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_657da9e3    n/a
token/         token         auth_token_14d7d5fd         token based credentials
```

Создадим yaml для ClusterRoleBinding

[Файл](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-vault/kubernetes-vault/vault-auth-service-account.yml)

Создадим Service Account vault-auth и применим ClusterRoleBinding
```
# Create a service account, 'vault-auth'
kubectl create serviceaccount vault-auth

# Update the 'vault-auth' service account
kubectl apply -f vault-auth-service-account.yml

```
Подготовим переменные для записи в конфиг кубер авторизации
```
export VAULT_SA_NAME=$(kubectl get sa vault-auth -o jsonpath="{.secrets[*]['name']}")
export SA_JWT_TOKEN=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data.token}" | base64 --decode; echo)
export SA_CA_CRT=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)
export K8S_HOST=$(more ~/.kube/config | grep server | awk '/http/ {print $NF}')
```

Запишем конфиг в vault
```
kubectl exec -it vault-0 -- vault write auth/kubernetes/config \
token_reviewer_jwt="$SA_JWT_TOKEN" \
kubernetes_host="$K8S_HOST" \
kubernetes_ca_cert="$SA_CA_CRT"

Success! Data written to: auth/kubernetes/config
```
Создадим [файл](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-vault/kubernetes-vault/otus-policy.hcl) политики

создадим политку и роль в vault 
```
kubectl cp otus-policy.hcl vault-0:/tmp
# Проверка 
kubectl exec -it -n default vault-0 -- ls -lh /tmp
total 8K
-rw-r--r--    1 vault    vault        125 Oct 25 10:29 otus-policy.hcl
-rw-r--r--    1 vault    vault        596 Oct 24 16:03 storageconfig.hcl 

kubectl exec -it vault-0 -- vault policy write otus-policy /tmp/otus-policy.hcl
# Сообщение
Success! Uploaded policy: otus-policy

kubectl exec -it vault-0 -- vault write auth/kubernetes/role/otus \
bound_service_account_names=vault-auth \
bound_service_account_namespaces=default policies=otus-policy ttl=24h
# Сообщение 
Success! Data written to: auth/kubernetes/role/otus
```
Проверим как работает авторизация
- Создадим под с привязанным сервис аккоунтом и установим туда curl и jq
```
kubectl run --generator=run-pod/v1 tmp --rm -i --tty --serviceaccount=vault-auth --image alpine:3.7
apk add curl jq

VAULT_ADDR=http://vault:8200
KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq
TOKEN=$(curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq '.auth.client_token' | awk -F\" '{print $2}')
```

Проверка
```
curl --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-ro/config
{"errors":["missing client token"]}

# Подставляем ранее выданный token
curl --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-ro/config
{"request_id":"4beeb90a-83cf-271c-4b2e-c26f1a98df4b","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}

curl --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-rw/config
{"request_id":"fcb30ee1-2d09-a5cc-6a34-45a0f80b1182","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}
``` 
```
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-ro/config
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-rw/config
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-rw/config1
```

```
/ # curl --request POST --data '{"BARR": "BAZZ"}' --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-rw/config | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    16    0     0  100    16      0   1000 --:--:-- --:--:-- --:--:--  1066
/ #
/ # curl --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-rw/config
{"request_id":"79a9399a-b097-27a5-aa8a-fb5dd373c611","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"BARR":"BAZZ"},"wrap_info":null,"warnings":null,"auth":null}
==============================

/ # curl --request POST --data '{"BARR": "BAZZ"}' --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-rw/config1 | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    16    0     0  100    16      0    727 --:--:-- --:--:-- --:--:--   727
/ #
/ # curl --header "X-Vault-Token:s.ddEdoJW5upyMKT4RHib5SoAo" $VAULT_ADDR/v1/otus/otus-rw/config1
{"request_id":"7492f7ff-aae5-c69d-9e4f-a02a4a355bd3","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"BARR":"BAZZ"},"wrap_info":null,"warnings":null,"auth":null}
```

В обоих случаях добавление успешное. Если нужно обновлять в [файл](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/blob/kubernetes-vault/kubernetes-vault/otus-policy.hcl) политики необходимо отредактировать
```
path "otus/otus-rw/*" {
capabilities = ["read", "create", "list", "updata"]
```

#### Use case использования авторизации через кубер
- Авторизуемся через vault-agent и получим клиентский токен
- Через consul-template достанем секрет и положим его в nginx
- Итог - nginx получил секрет из волта, не зная ничего про волт

Заберем репозиторий с примерами 
```
git clone https://github.com/hashicorp/vault-guides.git
cd vault-guides/identity/vault-agent-k8s-demo
```
- В каталоге configs-k8s скорректируйте конфиги с учетом ранее созданых ролей и секретов
- роверьте и скорректируйте конфиг example-k8s-spec.yml
- Скорректированные [конфиги](https://github.com/otus-kuber-2020-07/LinarNadyrov_platform/tree/kubernetes-vault/kubernetes-vault/configs-k8s) приложил к ДЗ


Проверка: 
- законнектится к поду nginx и вытащить оттуда index.html
- index.html приложить к ДЗ

