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
➜  ~
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
➜  ~
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

