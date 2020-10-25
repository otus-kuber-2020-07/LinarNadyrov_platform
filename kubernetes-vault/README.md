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

Проверка не удачная: 
```
kubectl exec -it -n default nginx-container -- bash
root@nginx-container:/#
root@nginx-container:/#
root@nginx-container:/# less /usr/s
sbin/  share/ src/
root@nginx-container:/# less /usr/share/nginx/html/
50x.html    index.html
root@nginx-container:/# cat /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### создадим CA на базе vault
Включим pki секретс
```
kubectl exec -it vault-0 -- vault secrets enable pki
kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki

kubectl exec -it vault-0 -- vault write -field=certificate pki/root/generate/internal \
common_name="exmaple.ru" ttl=87600h > CA_cert.crt
```
пропишем урлы для ca и отозванных сертификатов
```
kubectl exec -it vault-0 -- vault write pki/config/urls \
issuing_certificates="http://vault:8200/v1/pki/ca" \
crl_distribution_points="http://vault:8200/v1/pki/crl"
``` 

создадим промежуточный сертификат
```
kubectl exec -it vault-0 -- vault secrets enable --path=pki_int pki
kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki_int

kubectl exec -it vault-0 -- vault write -format=json pki_int/intermediate/generate/internal common_name="example.ru Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr
```

пропишем промежуточный сертификат в vault
```
kubectl cp pki_intermediate.csr vault-0:/home/vault/
kubectl exec -it vault-0 -- vault write -format=json pki/root/sign-intermediate csr=@/home/vault/pki_intermediate.csr format=pem_bundle ttl=43800h | jq -r '.data.certificate' > intermediate.cert.pem

kubectl cp intermediate.cert.pem vault-0:/home/vault/
kubectl exec -it vault-0 -- vault write pki_int/intermediate/set-signed certificate=@/home/vault/intermediate.cert.pem
```

Создадим и отзовем новые сертификаты
```
kubectl exec -it vault-0 -- vault write pki_int/roles/example-dot-ru allowed_domains="example.ru" allow_subdomains=true max_ttl="720h"
```
Создадим и отзовем сертификат
```
kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="gitlab.example.ru" ttl="24h"

===
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIURWTXwwGk74qq8J6+9YBugrFnYBAwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMDEwMjUxNDAwMTBaFw0yNTEw
MjQxNDAwNDBaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN10aTGSzjv1
nebE91VJVNH2E9vXrqf5re86ZKuAQRMtRp/inUnp93Pzpck5JWH9n6n65NNU7gcJ
Z0kBHv2UYzkKUjJOSg+zvDyXqeAU+LsAUJS1u+ZlPC8ttWnjz5OfnDMp2nt0g0qK
avUKmVyUQtUjF/XJrJ4aGzUyrgaGyKTUAT9Gk/3YgYPzxUkduJI4SkZ9aaMKMVml
x9FGJEYJHD60+lhmXsOHLsqJK0xFHQmCY1/xYTY3AA7DAyIf58yBuDKmzubyWjfx
iqiL90h9Zw2tnc53vZPDoA97+dM7PrY0whx77Yvk/3AUKqIV7fQ5mH9+bTLzIOTo
Yp0RunI25mcCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUr/p2B6aSoHi4V4rb65NbPtEi2NswHwYDVR0jBBgwFoAU
JHxTlPSkyeDkxMaMfQmrvS6wr/4wNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
ZiZxolvgCgqArahQ8cGapP16krn/1D2G20ShrCgoEHpYGuFE5lJsnTK5zIjeMEO8
umGJENQo/YG7bgNtDGJeOLOka+kaNuwVKbm5aSnX6F8H+GqKXTS42TtBCXzKn791
rSKAqhTeNM/GJ6hu1zMtjV87Yxehc/czWv8qBPd8CkOlcUd+kZJcVFFmdcjDPTiQ
tPjlMb/9gYcZgGREF5+m/C0Tbly0tHwUHSHoIK0I48bTdz4P9UuWPL7Ci2020k55
0ULKTuzJqS5rcCVf5gsrkLNoUQmVzgFl092gUJcpcZr993RS8Jgnz0g4OJeEPxxy
9mQTnG8gUUUcUyf1zxTfOw==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZzCCAk+gAwIBAgIUbOHlNLib6yUJJtmit0oMIy4fc24wDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIwMTAyNTE0MDIwMloXDTIwMTAyNjE0MDIzMlowHDEaMBgGA1UEAxMRZ2l0
bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDE
LmqCjVIif5jDDDYBM838NV6tdQMPtyMV89Dju8MOZqySBS2eKWoUSAXGh9LPIlml
UreoF0wFwUMy9d9jCZMjn4iN8nZQHowJcRch1pHoXvQepM2cc2WtIveSomHsqkzZ
dRfh5sNvnENLJ3JoNvmK9rpVkIbN70LCBK7cGHhCiFaGH4Xxv49Hmz2FljAG9+3p
UcC4VAJj2+xnkwSqnS1iylf7eootmceC3meiP2IgCKH8+tTPLeBervT9Zm8i219r
NdqnpS+Lsvcjd4skv6IhBAR5hJKoxfQxsORrxRwwJ//RgTf53Dyxps1Do1xczoCw
NQGH5hPgi8yjeNlsltivAgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQU8oizFNV3NpGRW5U5
srtjjRic9VAwHwYDVR0jBBgwFoAUr/p2B6aSoHi4V4rb65NbPtEi2NswHAYDVR0R
BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBAHvdJzKB
BOfNARo03zE7TeIJTLPAWLVgR8ExFT1eLwsowz/MzI48ej+wRz+UnPGdMNtnETBq
h1We8fWPp1pTfVaiG6XNzJg52gr8WHaqzi9TxIOLOgXUv4smcq7Pn6Pfe4+WCvlT
oRCR2CM4sZB9BHeVnazzSFzpgR1qKLjErap+SOF7pMz9mpFPqRcmnv9SefgYswbx
Fpmf4XBSOcdRZhl7eAE8UinMmo0SCjgewMjUK0RRvCOCXUU49v0CE85x6md1vtI2
A8sk8AjLzLKAjWU9ZmU/Fo8O040bDuL/oJzjts2UB1vNDgVNfRj5lOw1aAcXMXY5
qFGGye1kIRvw7D4=
-----END CERTIFICATE-----
expiration          1603720952
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIURWTXwwGk74qq8J6+9YBugrFnYBAwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMDEwMjUxNDAwMTBaFw0yNTEw
MjQxNDAwNDBaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN10aTGSzjv1
nebE91VJVNH2E9vXrqf5re86ZKuAQRMtRp/inUnp93Pzpck5JWH9n6n65NNU7gcJ
Z0kBHv2UYzkKUjJOSg+zvDyXqeAU+LsAUJS1u+ZlPC8ttWnjz5OfnDMp2nt0g0qK
avUKmVyUQtUjF/XJrJ4aGzUyrgaGyKTUAT9Gk/3YgYPzxUkduJI4SkZ9aaMKMVml
x9FGJEYJHD60+lhmXsOHLsqJK0xFHQmCY1/xYTY3AA7DAyIf58yBuDKmzubyWjfx
iqiL90h9Zw2tnc53vZPDoA97+dM7PrY0whx77Yvk/3AUKqIV7fQ5mH9+bTLzIOTo
Yp0RunI25mcCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUr/p2B6aSoHi4V4rb65NbPtEi2NswHwYDVR0jBBgwFoAU
JHxTlPSkyeDkxMaMfQmrvS6wr/4wNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
ZiZxolvgCgqArahQ8cGapP16krn/1D2G20ShrCgoEHpYGuFE5lJsnTK5zIjeMEO8
umGJENQo/YG7bgNtDGJeOLOka+kaNuwVKbm5aSnX6F8H+GqKXTS42TtBCXzKn791
rSKAqhTeNM/GJ6hu1zMtjV87Yxehc/czWv8qBPd8CkOlcUd+kZJcVFFmdcjDPTiQ
tPjlMb/9gYcZgGREF5+m/C0Tbly0tHwUHSHoIK0I48bTdz4P9UuWPL7Ci2020k55
0ULKTuzJqS5rcCVf5gsrkLNoUQmVzgFl092gUJcpcZr993RS8Jgnz0g4OJeEPxxy
9mQTnG8gUUUcUyf1zxTfOw==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAxC5qgo1SIn+Ywww2ATPN/DVerXUDD7cjFfPQ47vDDmaskgUt
nilqFEgFxofSzyJZpVK3qBdMBcFDMvXfYwmTI5+IjfJ2UB6MCXEXIdaR6F70HqTN
nHNlrSL3kqJh7KpM2XUX4ebDb5xDSydyaDb5iva6VZCGze9CwgSu3Bh4QohWhh+F
8b+PR5s9hZYwBvft6VHAuFQCY9vsZ5MEqp0tYspX+3qKLZnHgt5noj9iIAih/PrU
zy3gXq70/WZvIttfazXap6Uvi7L3I3eLJL+iIQQEeYSSqMX0MbDka8UcMCf/0YE3
+dw8sabNQ6NcXM6AsDUBh+YT4IvMo3jZbJbYrwIDAQABAoIBAQCThRfAfgZSPMKh
gNAnTU5KpdNA+elIav0uJ75fUTSW4qxHzS8FbL0A7TuykzX5XiotACtsccBP34jp
jCvjrDKBqhgkLTu8eYvyGaE8Z74mDyjg1ipqx/egHtgt4n9iWJkbOEqyKzWw+r87
hzknlpdFYMyzRM+pkY4QmTbn+FqOI+afyO5Ojmgq/WRwhQDYoi1Tm8Yme+eMGcJ8
bN+n5aIJsN8xBtEq645HMedyOMD1XVyo7uRINeuftVJs3NUI36LQOTiO5iighpAB
pl/tvDZuqjhXRqLLmjFpGeoC22ZYHlDj4p2UjNdb725G9dzJgkqPeemI6edm9jEz
4mN3XrpJAoGBAMom5ePErrrT+tfnQu0/u0vROxUo0Uqd8RuyZwU+xvycSUb88gwj
xDatGKSQo/hUfePWU3ZY42WmMregNjlNlPU8A2KKe7jyRR17IJoUPNcmkg3vRxc8
gng2ol9Aip2r6OfkW3SQnNj8762Z9WFj5Ki1ZQJPAh6gzxFhtSzvbyzDAoGBAPhw
XrUlAUtB8Qgz34a3aJH14YTtH7+PWGeg29UshwIQH7k4Tv8TfMg45EG5rOR6Sj10
ui2JsSycILKwx7QvcPCyV6TDARUHZ7UTTTwl6uN7emYyjxcp+V8XaF9Q90IW0mZG
53NS1eK2WPlNmP694rtVCWn2vSbmjoYVY14PgRWlAoGAXGHWYXa620qQsiZPoZik
gYuG0q6qpszzKNMo3W7JBRxvKG/kNcQKoWoNfvdS1+PU/FAwKD+K/CMtvlkLLrjT
wBbC/T+INwcQqt5gEcn52+EWkiOte2L9xO5C2gDm2BN+BquHWAfWhhthdRaM2wsJ
rkfnd1yf/VtEBg9++qAZUH0CgYEA2W8PlJ60iTdHSxSLV46B+O0n2XznQnKkvt2s
SOBVsNqHyUWc7eYSWfJg450r0jOtcigNIfnWlOJ4Q6wwvGShBigwSMVa1xrKC1K/
UBsnfrz9HSC212EnHbCQ6oskPDVZI6Z+vxIKnAdXy6m8c4ehPq9oM9N9LOSwbG2f
sB0FrvkCgYApWf74KPAfiaQ2HEF8KjqFfIB4WmDJDWQNqAa6/pzcJeYtrLv3AElq
RnCLs3HGlQalhyqO/61k9g7euPF5kvmQ8MQr1QazlPRGgymKPVGnGTn358GpSBen
xZoJ0U8lj2Psfo+w2T+3mG0mJc43SsvNUI/y/VJ+YGMM7KC0f6oY8A==
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       6c:e1:e5:34:b8:9b:eb:25:09:26:d9:a2:b7:4a:0c:23:2e:1f:73:6e
===

kubectl exec -it vault-0 -- vault write pki_int/revoke serial_number="6c:e1:e5:34:b8:9b:eb:25:09:26:d9:a2:b7:4a:0c:23:2e:1f:73:6e"

Key                        Value
---                        -----
revocation_time            1603634606
revocation_time_rfc3339    2020-10-25T14:03:26.43989494Z

```
