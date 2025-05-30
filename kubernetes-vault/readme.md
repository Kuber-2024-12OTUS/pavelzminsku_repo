# Выполнено ДЗ №

 - [ x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
Создаем k8s кластер в YC

![nodes](https://github.com/user-attachments/assets/723fd4d1-8d9c-4142-b2d6-163516556f9e)

Ставим consul и vault

` git clone https://github.com/hashicorp/consul-k8s.git .\install\consul\`
 
` git clone https://github.com/hashicorp/vault-helm.git .\install\vault  `
`
helm install consul -n consul --create-namespace .\install\consul\charts\consul\ -f .\values-consul.yaml`

![consul](https://github.com/user-attachments/assets/65a30530-5942-460e-949a-dc8871b94df8)

`helm install vault -n vault --create-namespace .\install\vault\ -f .\values-vault.yaml `

![vault](https://github.com/user-attachments/assets/560881cd-c13c-4fda-9f4f-992b02ec6502)

Проверяем:

![4 install_check](https://github.com/user-attachments/assets/b0dab049-d927-4fa0-82ab-e754f91bad08)

Инициализируем vault
```
kubectl exec -n vault vault-0 -- `
  vault operator init `
  -key-shares=1 `
  -key-threshold=1
```

![5_init](https://github.com/user-attachments/assets/ff3f287e-55da-4de2-b59b-c0aa2a51a7e7)

Получаем ключ и токен
```
Unseal Key 1: IIEstETyi4vy9YifpgNm0tJXRJcpKJtKqh14em7vrMg=

Initial Root Token: hvs.9JFICBS5iY8RJa0RtOeMggKR
```

Распечатываем:
$pods = "vault-0", "vault-1", "vault-2"
foreach ($pod in $pods) {
  kubectl exec -n vault $pod -- vault operator unseal IIEstETyi4vy9YifpgNm0tJXRJcpKJtKqh14em7vrMg=
}

Проверяем статус

` kubectl exec -n vault vault-0 -- vault status`
 
```
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.19.0
Build Date      2025-03-04T12:36:40Z
Storage Type    consul
Cluster Name    vault-cluster-69ba1d9c
Cluster ID      44b0c660-7589-8d80-e00f-2d4b46ef3ac1
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201 
HA Mode         active
Active Since    2025-05-29T23:08:09.348095208Z     
``` 

`kubectl exec -n vault vault-0 -- vault login hvs.9JFICBS5iY8RJa0RtOeMggKR`

```

kubectl exec -n vault vault-0 -- vault `
  secrets enable -path=otus kv-v2
```
![6_kv](https://github.com/user-attachments/assets/08f4fea3-6bab-40c9-9350-446a09e8759d)
`
kubectl exec -n vault vault-0 -- vault kv put otus/cred username=otus password=asajkjkahs`

![7_secret](https://github.com/user-attachments/assets/84e3ad9a-4258-4835-a18f-47dc93edd8fc)

kubectl exec -n vault vault-0 -- vault kv get otus/cred

![8_checl](https://github.com/user-attachments/assets/b1bd844b-705b-4e36-8591-9b97e74f85f6)

Применяем serviceaccount

`kubectl apply -f .\sa.yaml`

```
kubectl exec -n vault vault-0 -- vault auth enable kubernetes
Success! Enabled kubernetes auth method at: kubernetes/
```

kubectl cp otus-policy.hcl vault/vault-0:/tmp/
kubectl exec -n vault vault-0 -- vault policy write otus-policy /tmp/otus-policy.hcl

![9_policy](https://github.com/user-attachments/assets/2972e646-b8c0-4b85-bd22-a59b153817f9)

Создание роли в Kubernetes Auth

kubectl exec -n vault vault-0 -- vault write auth/kubernetes/role/otus `
  bound_service_account_names=vault-auth `
  bound_service_account_namespaces=vault `
  policies=otus-policy `
  ttl=24h

![10_check](https://github.com/user-attachments/assets/ad450f68-89ef-4eb3-9f32-3f05867d8afb)


# Проверка политик
`kubectl exec -n vault vault-0 -- vault policy list`

# Проверка роли
`kubectl exec -n vault vault-0 -- vault read auth/kubernetes/role/otus`

![10_policy](https://github.com/user-attachments/assets/5f4a3e74-19e7-4fcc-a637-78fa612ad425)

```
helm repo add external-secrets https://charts.external-secrets.io
"external-secrets" has been added to your repositories
 helm install external-secrets external-secrets/external-secrets -n vault 
NAME: external-secrets
LAST DEPLOYED: Fri May 30 02:38:25 2025
NAMESPACE: vault
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
external-secrets has been deployed successfully in namespace vault!


In order to begin using ExternalSecrets, you will need to set up a SecretStore
or ClusterSecretStore resource (for example, by creating a 'vault' SecretStore).

More information on the different types of SecretStores and how to configure them
can be found in our Github: https://github.com/external-secrets/external-secrets
```

Применяем манифест secretsore
```
 kubectl apply -f SecretStore.yaml
secretstore.external-secrets.io/vault-secret-store created

```
Применяем манифест externalsecred

```
 kubectl apply -f ExternalSecret.yaml
externalsecret.external-secrets.io/vault-secrets created

```
Проверяем

```
kubectl get secret -n vault otus-cred

NAME        TYPE     DATA   AGE
otus-cred   Opaque   2      18s
```

```
kubectl get secret -n vault otus-cred -o json | jq .data

{
  "password": "YXNhamtqa2Focw==",
  "username": "b3R1cw=="
}
```

```
kubectl get secret -n vault otus-cred -o json | jq -r .data.password | base64 -d

asajkjkahs%
```

```
kubectl get secret -n vault otus-cred -o json | jq -r .data.username | base64 -d

otus%
```


## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
