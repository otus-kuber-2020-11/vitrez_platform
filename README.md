# vitrez_platform
vitrez Platform repository

<details>
  <summary> Домашняя работа 11</summary>

## kubernetes-vault

Все домашнее задание выполнялось на локальном кластере, поэтому выполнено учитывая его специфику и ресурсы. Все объекты будем создавать в неймспейсе vault.

#### Установка и настройка Vault под задачи Kubernetes 

- Добавим репозиторий hashicorp, кастомизируем переменные values и установим чарты vault и consul:
```
helm repo add hashicorp https://helm.releases.hashicorp.com

helm show values hashicorp/consul > consul.values.yaml
helm upgrade --install consul hashicorp/consul -f consul.values.yaml --namespace consul --create-namespace

helm show values hashicorp/vault > vault.values.yaml
helm upgrade --install vault hashicorp/vault -f vault.values.yaml --namespace vault --create-namespace
```

- Посмотрим статус релиза helm:
```
$ helm status vault -n vault
NAME: vault
LAST DEPLOYED: Sat May 15 20:59:36 2021
NAMESPACE: vault
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing HashiCorp Vault!
```

- Смотрим состояние подов vault:
```
$ kubectl get pod -n vault 
NAME                                        READY   STATUS    RESTARTS   AGE
pod/vault-0                                 0/1     Running   0          6m25s
pod/vault-agent-injector-77fbb4d4f8-5n762   1/1     Running   0          6m25s
```

- Посмотрим логи пода vault:
```
$ kubectl logs pod/vault-0 -n vault 
==> Vault server configuration:

             Api Address: http://10.244.0.172:8200
                     Cgo: disabled
         Cluster Address: https://vault-0.vault-internal:8201
              Go Version: go1.15.10
              Listener 1: tcp (addr: "[::]:8200", cluster address: "[::]:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level: info
                   Mlock: supported: true, enabled: false
           Recovery Mode: false
                 Storage: file
                 Version: Vault v1.7.0
             Version Sha: 4e222b85c40a810b74400ee3c54449479e32bb9f

==> Vault server started! Log data will stream in below:

2021-05-15T18:00:00.771Z [INFO]  proxy environment: http_proxy= https_proxy= no_proxy=
2021-05-15T18:00:05.883Z [INFO]  core: security barrier not initialized
2021-05-15T18:00:05.884Z [INFO]  core: seal configuration missing, not initialized
2021-05-15T18:00:10.867Z [INFO]  core: security barrier not initialized
2021-05-15T18:00:10.868Z [INFO]  core: seal configuration missing, not initialized
2021-05-15T18:00:15.874Z [INFO]  core: security barrier not initialized
2021-05-15T18:00:15.874Z [INFO]  core: seal configuration missing, not initialized
2021-05-15T18:00:20.879Z [INFO]  core: security barrier not initialized
2021-05-15T18:00:20.879Z [INFO]  core: seal configuration missing, not initialized
2021-05-15T18:00:25.905Z [INFO]  core: security barrier not initialized
2021-05-15T18:00:25.905Z [INFO]  core: seal configuration missing, not initialized
2021-05-15T18:00:30.918Z [INFO]  core: security barrier not initialized
```

- Выполним инициализацию vault (через любой под vault'a):
```
$ kubectl exec -n vault -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1
Unseal Key 1: 9xE8rrZaGpwMWptVESte2HIU/e9yJORHe7XdxdwDFko=

Initial Root Token: s.jFViiLu5x6mVGd4dBZsG9rtQ

Vault initialized with 1 key shares and a key threshold of 1.
```

- Посмотрим логи пода vault:
```
$ kubectl logs vault-0 -n vault
...
2021-05-15T18:08:24.448Z [INFO]  core: security barrier initialized: stored=1 shares=1 threshold=1
2021-05-15T18:08:24.492Z [INFO]  core: post-unseal setup starting
2021-05-15T18:08:24.536Z [INFO]  core: loaded wrapping token key
2021-05-15T18:08:24.536Z [INFO]  core: successfully setup plugin catalog: plugin-directory=
2021-05-15T18:08:24.538Z [INFO]  core: no mounts; adding default mount table
2021-05-15T18:08:24.559Z [INFO]  core: successfully mounted backend: type=cubbyhole path=cubbyhole/
2021-05-15T18:08:24.559Z [INFO]  core: successfully mounted backend: type=system path=sys/
2021-05-15T18:08:24.559Z [INFO]  core: successfully mounted backend: type=identity path=identity/
2021-05-15T18:08:24.622Z [INFO]  core: successfully enabled credential backend: type=token path=token/
2021-05-15T18:08:24.624Z [INFO]  core: restoring leases
2021-05-15T18:08:24.625Z [INFO]  rollback: starting rollback manager
2021-05-15T18:08:24.628Z [INFO]  expiration: lease restore complete
2021-05-15T18:08:24.648Z [INFO]  identity: entities restored
2021-05-15T18:08:24.648Z [INFO]  identity: groups restored
2021-05-15T18:08:24.649Z [INFO]  core: usage gauge collection is disabled
2021-05-15T18:08:24.663Z [INFO]  core: post-unseal setup complete
2021-05-15T18:08:24.697Z [INFO]  core: root token generated
2021-05-15T18:08:24.697Z [INFO]  core: pre-seal teardown starting
2021-05-15T18:08:24.697Z [INFO]  rollback: stopping rollback manager
2021-05-15T18:08:24.697Z [INFO]  core: pre-seal teardown complete
```

- Посмотрим текущий статус vault:
```
$ kubectl -n vault exec -it vault-0 -- vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       1
Threshold          1
Unseal Progress    0/1
Unseal Nonce       n/a
Version            1.7.0
Storage Type       file
HA Enabled         false
command terminated with exit code 2
```

- Распечатаем vault (распечатать нужно каждый под\сервер) и посмотрим теперь его статус:
```
$ kubectl -n vault exec -it vault-0 -- vault operator unseal
9xE8rrZaGpwMWptVESte2HIU/e9yJORHe7XdxdwDFko=

$ kubectl -n vault exec -it vault-0 -- vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.7.0
Storage Type    file
Cluster Name    vault-cluster-04036513
Cluster ID      e5c155ff-f663-3e14-aefc-9f9b8453bc38
HA Enabled      false

$ kubectl get pod -n vault 
NAME                                        READY   STATUS    RESTARTS   AGE
pod/vault-0                                 1/1     Running   0          20m
pod/vault-agent-injector-77fbb4d4f8-5n762   1/1     Running   0          20m
```

- Залогинимся в vault с помощью токена root (Root Token):
```
$ kubectl -n vault exec -it vault-0 -- vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.jFViiLu5x6mVGd4dBZsG9rtQ
token_accessor       4RqJTOe1sc7nL0sbFPfg5rgf
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

- Выведем список включенных типов аутентификации:
```
$ kubectl -n vault exec -it vault-0 -- vault auth list
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_c950ec92    token based credentials
```

- Включим движок секретов типа kv (key-value) по определенному пути: 
```
$ kubectl -n vault exec -it vault-0 -- vault secrets enable --path=otus kv
$ kubectl -n vault exec -it vault-0 -- vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_55551331    per-token private secret storage
identity/     identity     identity_7be9334c     identity store
otus/         kv           kv_9f92a378           n/a
sys/          system       system_6e089ca4       system endpoints used for control, policy and debugging
```
Кстати, если в команде включения добавить параметр '-version=2', то движок kv будет версии 2, который позволит версионировать секреты, т.е. можно будет изменять значения секрета и читать их разные версии.

- Заведем секреты:
```
$ kubectl -n vault exec -it vault-0 -- vault kv put otus/otus-ro/config username='otus' password='asajkjkahs'
$ kubectl -n vault exec -it vault-0 -- vault kv put otus/otus-rw/config username='otus' password='asajkjkahs'

$kubectl -n vault exec -it vault-0 -- vault kv get otus/otus-rw/config
====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```

- Включим аутентификацию через k8s:
```
$ kubectl -n vault exec -it vault-0 -- vault auth enable kubernetes
$ kubectl -n vault exec -it vault-0 -- vault auth list
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_80dec093    n/a
token/         token         auth_token_c950ec92         token based credentials
```

- Создадим Service Account с именем vault-auth и применим ClusterRoleBinding из файла [манифеста](kubernetes-vault/vault-auth-service-account.yml)
```
$ kubectl create serviceaccount vault-auth -n vault
$ kubectl apply --filename vault-auth-service-account.yml
```
Насколько я понял, кластерная роль system:auth-delegator позволит SA 'vault-auth', от лица которой будет ходить в API K8s наш Vault, проверять токены других SA, обращающихся за секретами к Vault. По их токенам он найдет их имена, неймспейсы и свяжет с ролями внутри Vault.

 
- Подготовим переменные для записи в конфиг аутентификации kubernetes:
```
$ export VAULT_SA_NAME=$(kubectl -n vault get sa vault-auth -o jsonpath="{.secrets[*]['name']}")
$ export SA_JWT_TOKEN=$(kubectl -n vault get secret $VAULT_SA_NAME -o jsonpath="{.data.token}" | base64 --decode; echo)
$ export SA_CA_CRT=$(kubectl -n vault get secret $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)
$ export K8S_HOST=$(kubectl cluster-info | grep 'Kubernetes master' | awk '/https/ {print $NF}' | sed 's/\x1b\[[0-9;]*m//g')
```
Кстати, конструкция (sed 's/\x1b\[[0-9;]*m//g') удаляет непечатные символы ANSI-подсветки кода.

- Запишем конфиг аутентификации kubernetes в vault:
```
$ kubectl -n vault exec -it vault-0 -- vault kv put auth/kubernetes/config \
token_reviewer_jwt="$SA_JWT_TOKEN" \
kubernetes_host="$K8S_HOST" \
kubernetes_ca_cert="$SA_CA_CRT"
```

- Теперь создадим политку в vault из [файла](kubernetes-vault/otus-policy.hcl)
```
$ kubectl -n vault cp otus-policy.hcl vault-0:/home/vault/
$ kubectl -n vault exec -it vault-0 -- vault policy write otus-policy /home/vault/otus-policy.hcl
```

- Создадим роль в vault, связав в ней service account некого пода и политику. В ДЗ мы должны использовать тот же SA, что и для аутентификации vault в K8s, но для внесения ясности о том, что на практике это должны/могут быть другие SA, я решил использовать Service Account по имени default.
```
$ kubectl -n vault exec -it vault-0 -- vault write auth/kubernetes/role/otus \
bound_service_account_names=default \
bound_service_account_namespaces=vault \
policies=otus-policy \
ttl=24h
```

- Создадим под с привязанным сервис аккаунтом и установим туда curl и jq:
```
$ kubectl -n vault run tmp --rm -i --tty --serviceaccount=default --image alpine:3.7
apk add curl jq
```

- Находясь в консоли пода, получим клиентский токен vault, используя k8s-токен нашего service account:
```
VAULT_ADDR=http://vault:8200
KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq
{
"request_id": "5738c554-263a-ca67-00af-644d908120a7",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": null,
  "wrap_info": null,
  "warnings": null,
  "auth": {
    "client_token": "s.iC6DB3t0UgF9rQ87yIvJcNLz",
    "accessor": "Bv53gdRwZS9tWs6yEOyXAyLK",
    "policies": [
      "default",
      "otus-policy"
    ],
    "token_policies": [
      "default",
      "otus-policy"
    ],
    "metadata": {
      "role": "otus",
      "service_account_name": "default",
      "service_account_namespace": "vault",
      "service_account_secret_name": "default-token-6z4zk",
      "service_account_uid": "ca6d74e6-848e-462f-82e0-f2cc476cee27"
    },
    "lease_duration": 86400,
    "renewable": true,
    "entity_id": "26ba43aa-4b9f-7de3-10f8-aef3b9c4ba4b",
    "token_type": "service",
    "orphan": true
  }
}

TOKEN=$(curl -k -s --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq '.auth.client_token' | awk -F\" '{print $2}')
```

- Теперь, используя клиентский токен vault, проверим чтение ранее созданных секретов:
```
/ # curl --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-ro/config
{"request_id":"8bf2eae1-a295-5542-7613-51cb314dab12","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}

/ # curl --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config
{"request_id":"dd5b0b4c-4d64-e5d7-ee40-35a79febd69f","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}
```

- Также проверим запись секретов:
```
/ # curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-ro/config
{"errors":["1 error occurred:\n\t* permission denied\n\n"]}

/ # curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config
{"errors":["1 error occurred:\n\t* permission denied\n\n"]}

/ # curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config1
```
Почему мы смогли записать otus-rw/config1 но не смогли otus-rw/config?    
Ответ:    
путь otus-ro/config - только для чтения;  
путь otus-rw/config - у нас отсутствует capability "update", добавим его;  
путь otus-rw/config1 - capability "create" у нас есть;  

#### Пример получения секрета из Vault в Kubernetes

- Опишем что будет происходить "под капотом" в примере:
  - Авторизуемся через vault-agent и получим клиентский токен
  - Через consul-template достанем секрет и положим его в nginx
  - Все действия проводятся через init-контейнер, в итоге nginx получит секрет, ничего не зная про vault

- Скопируем из [репозитория](https://github.com/hashicorp/vault-guides.git) манифесты [configmap](kubernetes-vault/configs-k8s/configmap.yaml) и [пода](kubernetes-vault/configs-k8s/example-k8s-spec.yaml)
- Отредактируем их с учетом ранее созданых ролей и секретов
- Применим их и вытащим из пода nginx файл [index.html](kubernetes-vault/index.html) с секретами:
```
$ kubectl apply -f ./configs-k8s/configmap.yaml
$ kubectl apply -f ./configs-k8s/example-k8s-spec.yaml --record
$ kubectl -n vault exec -it vault-agent-example -- cat /usr/share/nginx/html/index.html | tee index.html
<html>
<body>
<p>Some secrets:</p>
<ul>
<li><pre>username: otus</pre></li>
<li><pre>password: asajkjkahs</pre></li>
</ul>

</body>
</html>
```

#### Работа с сертификатами на базе Vault

- Cоздадим CA на базе vault:
```
$ kubectl -n vault exec -it vault-0 -- vault secrets enable pki
Success! Enabled the pki secrets engine at: pki/
$ kubectl -n vault exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki
Success! Tuned the secrets engine at: pki/
$ kubectl -n vault exec -it vault-0 -- vault write -field=certificate pki/root/generate/internal common_name="example.ru" ttl=87600h > CA_cert.crt
```

- Пропишем URL's для CA и CRL (списка отозванных сертификатов):
```
$ kubectl -n vault exec -it vault-0 -- vault write pki/config/urls \
> issuing_certificates="http://vault:8200/v1/pki/ca" \
> crl_distribution_points="http://vault:8200/v1/pki/crl"
Success! Data written to: pki/config/urls
```

- Создадим CSR для сертификата промежуточного CA:
```
kubectl -n vault exec -it vault-0 -- vault secrets enable --path=pki_int pki
kubectl -n vault exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki_int
kubectl -n vault exec -it vault-0 -- vault write -format=json \
pki_int/intermediate/generate/internal \
common_name="example.ru Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr
```

- Выпустим и подпишем на основе CSR сертификат промежуточного CA и положим его в vault:
```
$ kubectl -n vault cp pki_intermediate.csr vault-0:/home/vault/
$ kubectl -n vault exec -it vault-0 -- vault write -format=json pki/root/sign-intermediate \
csr=@/home/vault/pki_intermediate.csr \
format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > intermediate.cert.pem

$ kubectl -n vault cp intermediate.cert.pem vault-0:/home/vault/
$ kubectl -n vault exec -it vault-0 -- vault write pki_int/intermediate/set-signed \
certificate=@/home/vault/intermediate.cert.pem
```

- Создадим роль для выдачи сертификатов:
```
$ kubectl -n vault exec -it vault-0 -- vault write pki_int/roles/example-dot-ru \
allowed_domains="example.ru" allow_subdomains=true max_ttl="720h"
```

- Создадим новый сертификат:
```
$ kubectl -n vault exec -it vault-0 -- vault write pki_int/issue/example-dot-ru \
> common_name="gitlab.example.ru" ttl="24h"
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUJ7rA66L/3W/RGMESqe7nlnmWs9EwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMTA1MjQyMDI4MjdaFw0yNjA1
MjMyMDI4NTdaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMCyVF5jW9cU
rnjHmJean2VVZZuUaXE7uWqTjSp/mf3NkqjHSRhfclrH7zM1D4YOk6T2lXm/PjPZ
XNSb0wdL3DnrDuarm/hmVgAIXanjtrCpqIxSaK6/yl968v2UOkC1z50JsZNn1H59
1bY6e4fPFk1S5wndfUOP/jyxWqqTO5p89Htv0nTGsnY4TptlPtkhoASOuW7Foj2o
K63UROcdTUs7cRe4HjppTmQBWhnJKTU8tUAgLsuVcwimlqMpFl7byc5pR0Xhop3k
0aSmXHkDnLtgfnZ23y+JMT2xPEsai9YHQcHvrNNYADVIlt7ONfTpK939/PNNzPyS
Yzum35FuZLsCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQU/A/ib5auJ0Zca+gbtdscIYV8qwswHwYDVR0jBBgwFoAU
davBHMKuSNhMp7JLJmDAdR1pkhAwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
nz3lMR0J8+xzQZm4xtjLShUch+k621Ns0OmjG9/w4Tn3ZJ3AGWz7GstC5Nn83ZVO
+Y2SMfwHny+lAa2NL9JR4qEIpjFkbCVAlGE2DEIjcBLdCc1mMB0KJBp4USMeO93V
BnBT9ehbEqqrZSR4n3CMcpzckYPtd4f/8fJL0ctFqXO02MdFEA708EYa6X2wEcqN
+pZ5QEzvyIWwSv4ttMZUu5B731yjVBLNyyhLiPOabgM22U9eZm+hploP2I+c55vM
+omc1qoOefeYv4LNirb2i9RF4bCFuBezFGH9N9it2YUyE9wvN8d758CJlvyQpObF
sdwT7XrETI7oBRQ5mzndqA==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZzCCAk+gAwIBAgIUN858COmY+O1hsDr8DIW7NGLw3bkwDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIxMDUyNDIwNDMxM1oXDTIxMDUyNTIwNDM0M1owHDEaMBgGA1UEAxMRZ2l0
bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDf
mUYWzjUHdHbjktZarO/bmW4hj8UbPX7MaFRlMxFYFGchCkwSL7004D9IjN2xf7fy
YedOTHMWbMPnAnkoq2/eLmjbpsNrM/+94+m9LONU/XCRzISfMzea3mkk82PFLd/W
N1DMB1NZnekTFb4Fo9loR60ovpx2qsgLKtfb32QqVhiEf8tmxb4fFhn+WFsEIDfp
OJK4Y8xTq9Q8oz3CPrSLzckfHEGMtvUv6ce9V+mnLUMGbQ/uQvCnkzD+hSDswxSK
+UA7BW6MFCTnjRYhRe58HQCVXVRWcthqjJCrE/KoOe9IIiBZoXV1yfsK8C8YGi7B
f0K1qd6vKq2Yyd/vlWY7AgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQUaYYwenjdCc2by8lL
pnSoIDMtRMMwHwYDVR0jBBgwFoAU/A/ib5auJ0Zca+gbtdscIYV8qwswHAYDVR0R
BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBAAxAhJF3
VpfLntZGULxQnnT04Ail3bQxnI+wFmzMBaGFnCYX90wnuI1KHIXedLoxGTA28JQJ
vYJ7ciA518unN1MXXnWWZRrPKwF5DFr69aF6v4vPMLPSBWC9hjeFav+K07wXzcJE
PDvHx6i2jiQ73jEeyrLZ4TKgrbS0cpWPzKdAxkGZVy/KKdeF48hc76V5vlnzWIsd
4jHN7MIJZ2aJDJO5qSgv3llL6Ci5TSH1ucNe0K4Y7RyG1BTQLTnLqm7LlLoC4Tbe
8hzi8UXn9XEjwUd/qIGhLEuB+8XXNnZXuObkKFCPI5gklFIOasPMrkHDsWAeRfFa
sMeJ3J9YUB1Wmt4=
-----END CERTIFICATE-----
expiration          1621975423
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUJ7rA66L/3W/RGMESqe7nlnmWs9EwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMTA1MjQyMDI4MjdaFw0yNjA1
MjMyMDI4NTdaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMCyVF5jW9cU
rnjHmJean2VVZZuUaXE7uWqTjSp/mf3NkqjHSRhfclrH7zM1D4YOk6T2lXm/PjPZ
XNSb0wdL3DnrDuarm/hmVgAIXanjtrCpqIxSaK6/yl968v2UOkC1z50JsZNn1H59
1bY6e4fPFk1S5wndfUOP/jyxWqqTO5p89Htv0nTGsnY4TptlPtkhoASOuW7Foj2o
K63UROcdTUs7cRe4HjppTmQBWhnJKTU8tUAgLsuVcwimlqMpFl7byc5pR0Xhop3k
0aSmXHkDnLtgfnZ23y+JMT2xPEsai9YHQcHvrNNYADVIlt7ONfTpK939/PNNzPyS
Yzum35FuZLsCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQU/A/ib5auJ0Zca+gbtdscIYV8qwswHwYDVR0jBBgwFoAU
davBHMKuSNhMp7JLJmDAdR1pkhAwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
nz3lMR0J8+xzQZm4xtjLShUch+k621Ns0OmjG9/w4Tn3ZJ3AGWz7GstC5Nn83ZVO
+Y2SMfwHny+lAa2NL9JR4qEIpjFkbCVAlGE2DEIjcBLdCc1mMB0KJBp4USMeO93V
BnBT9ehbEqqrZSR4n3CMcpzckYPtd4f/8fJL0ctFqXO02MdFEA708EYa6X2wEcqN
+pZ5QEzvyIWwSv4ttMZUu5B731yjVBLNyyhLiPOabgM22U9eZm+hploP2I+c55vM
+omc1qoOefeYv4LNirb2i9RF4bCFuBezFGH9N9it2YUyE9wvN8d758CJlvyQpObF
sdwT7XrETI7oBRQ5mzndqA==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA35lGFs41B3R245LWWqzv25luIY/FGz1+zGhUZTMRWBRnIQpM
Ei+9NOA/SIzdsX+38mHnTkxzFmzD5wJ5KKtv3i5o26bDazP/vePpvSzjVP1wkcyE
nzM3mt5pJPNjxS3f1jdQzAdTWZ3pExW+BaPZaEetKL6cdqrICyrX299kKlYYhH/L
ZsW+HxYZ/lhbBCA36TiSuGPMU6vUPKM9wj60i83JHxxBjLb1L+nHvVfppy1DBm0P
7kLwp5Mw/oUg7MMUivlAOwVujBQk540WIUXufB0AlV1UVnLYaoyQqxPyqDnvSCIg
WaF1dcn7CvAvGBouwX9CtaneryqtmMnf75VmOwIDAQABAoIBAAFLBp+9I4tefg2E
3N57X4u6kGt7RF2K9n/CHrLTH8eNnqcPQy9bvVFf9p25ytJq9apeLJNEV+oKSPu+
BOtaSnRTemHCziCBlXoIpmJkrw/fw1Xkg+PTzP+FR8Bh8/LA+Clp+nqjlDTRd/aX
SpkHwIsc1wCEUa1SAYQnBEaOPSsNf6RXZzZakJYAp3gAjj4opCywlncy15b2GY1E
FSJj2QU5/uzoLjzvhoYVGrJR4+4rUgRG+USZ5rOCw5K9IwzFTIuQ6veBonnkNYGr
a2EOvD9T2nePnMYxLRmCYd4ShnGcpHpeKaonYtgQMEAiKSSOyhQWaUYWxgg9kBSN
YIrXUCECgYEA8hNq4kBEumKmBgj5zwpjJ7hXmmHKRibHpNjjQQ7emqC9c/b/JNxH
h6W99uNRWHHcrORnRVyy4jiPTvNU6EjsnPWYyUxi53qceKqlojlY3iNw1vOTxDiO
4pRFOXK1gjK+W6MZh9tjeJJEMd4+ir38EPIp7BcVaWMsFxfp6XVphHECgYEA7HXH
iLk6zqq2rt8QBxo9LQMboVkX+nzOOUZGzyAjCE5YgSCcrSjTwumpmdpN/0SuWZz2
kBcwnK2E1uWO7bnkcD6a3qqusp4tGmuPnByI47kyCvTQxEJmIJqDM8oA4H40l7DC
U2HWpMZ0r/cN/mDrBu1l+xghjEDJAn/j86DFO2sCgYA93rScquxl7rycIkMmpXL+
PeE19fRqxZKVEVHT2OcQAjEpqGFBnIMzqirJJQQvZLqP/bhfQ/f8VZRbC1oSHEFN
RIAOQtWsb+v58zNuKNYLwGcgqRSFPCdYxaiDrEuzwSBh72ehD3N253tCe5jkgPYh
pqMMUkIIs24bYONJ5dZYIQKBgQDiHLpiXqYCbDJmxD0iXY/0VA1+26BXUjMth6s8
c0GstqZhTBsmZm0g7KnWym9dU4LZhIQuQ06j9DWb/UYQw3rTbrpPhK2rdiAxLHvW
T18DS9uzqGld0xSvxrEBu//crDKEf21DqMJFLNT2U2vZPTphlG+5jVi/MlBFCKCl
Hq6b4wKBgBPhkWzS1RXf+3ynt7TEVptXeNMunUwJ7bn2e48T/4x/1EpzDJGen2GV
7F01axqcOgLrj8pYQv8MXAevTH1TiMFReL9Ch6i1yT4//WW0GcysEC+PrV/WtR2A
7El8P1YLac1OKCwTQCwcgMQJJ3mCFRSXI4Hyfz3WkVpAtOQVemQ4
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       37:ce:7c:08:e9:98:f8:ed:61:b0:3a:fc:0c:85:bb:34:62:f0:dd:b9
```

- Отзовем только что выпущенный сертификат:
```
$ kubectl -n vault exec -it vault-0 -- vault write pki_int/revoke \
> serial_number="37:ce:7c:08:e9:98:f8:ed:61:b0:3a:fc:0c:85:bb:34:62:f0:dd:b9"
Key                        Value
---                        -----
revocation_time            1621889539
revocation_time_rfc3339    2021-05-24T20:52:19.878774141Z
```

</details>

<details>
  <summary> Домашняя работа 9</summary>

## kubernetes-logging

Все домашнее задание выполнялось на локальном кластере, поэтому выполнено учитывая его специфику. 

#### Установка и мониторинг EFK стека
- Установлены EFK-стэк (ElasticSearch, Fluent Bit, Kibana) и nginx-ingress-controller с помощью Helm-чартов и своих values.  
Из-за ограниченности системных ресурсов пришлось запускать кластер Elasticsearch всего из одной ноды и регулировать запрашиваемые ресурсы во всех чартах.  
По этой же причине не использовал taint\tolerations.

- Воспользовались во fluent-bit фильтром Modify, который позволил удалить из логов "лишние" ключи time и @timestamp.  
Настройку проводим через параметры в fluent-bit.values.yaml

- Установили Grafana, Prometheus и prometheus-operator из helm-чарта kube-prometheus-stack.  
Добавили prometheus exporter для ElasticSearch из [helm-чарта](https://github.com/justwatchcom/elasticsearch_exporter)  
В графану залит популярный дашборд для мониторинга elasticsearch. Рассмотрели его ключевые метрики.  

- Логи контроллера Nginx Ingress.  
Т.к. у меня Fluent Bit устанавливается сразу на все ноды кластера, то проблем с поиском логов nginx-ingress-controller не возникло.  
Поменяем формат логов у нашего nginx-ingress на формат JSON. Для этого изменим конфигурацию через nginx-ingress.values.yaml добавив ключи log-format-escape-json и log-format-upstream.  
Создали в Kibana визуализации для отображения запросов к nginx-ingress со статусами:  
200-299  
300-399  
400-499  
500+  
На их базе создали дашборд kibana и выгрузили в формате [json](kubernetes-logging/export.ndjson)  

#### Установка Loki: сбор и визуализация логов в Grafana.
- Установили Loki и Promtail с помощью [helm-чарта](https://grafana.github.io/loki/charts)  
Изменили конфигурацию prometheus-operator таким образом,чтобы datasource графаны для Loki создавался сразу после установки оператора.  
Выложил итоговый [values](kubernetes-logging/kube-prometheus-stack.values.yaml) для prometheus-operator.  

- Изменили [values](kubernetes-logging/nginx-ingress.values.yaml) для контроллера nginx-ingress таким образом, чтобы он начал отдавать метрики в формате prometheus:
```
metrics:
    enabled: true
    service:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "10254"
```

и создавал объект serviceMonitor для prometheus-operator:
```
serviceMonitor:
      port: 10254
      enabled: true
```

- Cоздали дашборд в графане, на котором одновременно вывели метрики контроллера nginx-ingress и его логи:
  - добавлены переменные для возможности выбора на дашборде инстанса nginx-ingress, неймспейса и класса контроллера
  - добавлена панель с графиком объема всех запросов к контроллеру nginx-ingress
  - добавлена панель с графиком числа неуспешных запросов (4хх-5хх)
  - добавлена панель с логами контроллера
Сам дашборд выгрузилив в формате [JSON](kubernetes-logging/nginx-ingress.json).

#### Задание со * [Audit logging]
Для сбора логов аудита кластера (api-server) нам нужно вначале включить их создание\ведение на кластере, а потом сбор их через fluentbit.
Теперь по шагам:
1) Создаем файлик с политикой аудита [audit-policy.yaml](kubernetes-logging/audit-policy.yaml) (для простоты включил сохранение метаданных всех событий) и кладем его на ноду с kube-apiserver.
Теперь редактируем параметры запуска kube-apiserver через правку манифеста его StaticPod на мастер-ноде. Добавляем следующие блоки:

```
# vi /etc/kubernetes/manifests/kube-apiserver.yaml

...
spec:
  containers:
  - command:
    - kube-apiserver
    - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
    - --audit-log-path=/var/log/kube-apiserver-audit.log
    - --audit-log-maxage=1
    - --audit-log-maxbackup=2
    - --audit-log-maxsize=20


...
volumeMounts:
  - mountPath: /etc/kubernetes/audit-policy.yaml
    name: audit
    readOnly: true
  - mountPath: /var/log/kube-apiserver-audit.log
    name: audit-log
    readOnly: false

...
volumes:
- name: audit
  hostPath:
    path: /etc/kubernetes/audit-policy.yaml
    type: File

- name: audit-log
  hostPath:
    path: /var/log/kube-apiserver-audit.log
    type: FileOrCreate
```

2) Редактируем fluentbit.values.yaml и добавляем секцию:
``` 
audit:
  enable: true
  input:
    path: /var/log/kube-apiserver-audit.log
```

#### Задание со * [Host logging]
Для сбора логов с виртуальных машин, где запущен K8s (его сервисы kubelet) редактируем fluentbit.values.yaml и добавляем секцию:
```
input:
  systemd:
    enabled: true
```

</details>

<details>
  <summary> Домашняя работа 8</summary>

  ## kubernetes-monitor

Выбран 4 вариант сложности: поставить prometheus-operator при помощи helm3.

- Helm-чарт для prometheus-operator называется kube-prometheus-stack. Переопределяем нужные нам параметры в values.yaml и ставим его:
```
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -f kube-prometheus-stack/values.yaml --namespace monitoring --create-namespace
```
- Добавляем адреса Ingress, указанные нами в values, в файл C:\Windows\System32\drivers\etc\hosts чтобы открывать их в браузере:
```
172.16.255.2		grafana.k8s.local
172.16.255.2		prometheus.k8s.local
172.16.255.2		alertmanager.k8s.local
```

- Т.к. кластер K8s у меня локальный (на базе k1s), то возникли некоторые проблемы с мониторингом:
  - не мониторятся kube-proxy на нодах
  - не мониторится etcd

  Решим эти проблемы.

1) По дефолту kube-proxy отдает метрики только через localhost.
Чтобы prometheus-operator смог забирать метрики нужно чтобы kube-proxy слушал адрес 0.0.0.0. Для этого необходимо поправить его настройки в ConfigMap:
```
$ kubectl edit configmaps kube-proxy -n kube-system
```
  устанавливаем следующий параметр:
```
metricsBindAddress: "0.0.0.0:10249"
```
  сохраняем и перзапускаем поды DaemonSet kube-proxy:
```
$ kubectl rollout restart daemonset kube-proxy -n kube-system
```
2) Чтобы снимать метрики с etcd необходима двусторонняя аутентификация по mtls.
Создадим сертификаты для Prometheus чтобы он мог успешно подключаться к etcd.

  для этого залогинимся на master-ноду (там развернут инстанс etcd) и скопируем клиентские сертификаты в новый secret:
```
kubectl create secret generic etcd-client-cert -n monitoring \
  --from-literal=etcd-ca="$(cat /etc/kubernetes/pki/etcd/ca.crt)" \
  --from-literal=etcd-client="$(cat /etc/kubernetes/pki/etcd/healthcheck-client.crt)" \
  --from-literal=etcd-client-key="$(cat /etc/kubernetes/pki/etcd/healthcheck-client.key)"
```
  теперь отредактируем файл переменных для helm-чарта, добавив имя secret в prometheusSpec:
```
prometheusSpec:
  secrets:
      - etcd-client-cert
```
  перенакатим чарт:
```
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -f kube-prometheus-stack/values.yaml
```
  Все, проблемные сервисы завсветились в Prometheus:

  kube-proxy:
![screen1](kubernetes-monitoring/images/prometheus_kube-proxy.png)

  etcd:
![screen2](kubernetes-monitoring/images/prometheus_etcd.png)

- Развернут Deployment с нашим приложением в виде контейнера nginx и sidecar-контейнера [nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)
В конфигурации nginx через ConfigMap включаем отдачу метрик:
```
location = /basic_status {
stub_status;
}
``` 
Экспортеру передается аргумент для сбора метрик:
```
args: [ "-nginx.scrape-uri", "http://localhost:8000/basic_status" ]
```
Также развернуты [Service](kubernetes-monitoring/web-svc-headless.yaml) и [Ingress](kubernetes-monitoring/ingress.yaml) для нашего приложения.

- Создан объект ServiceMonitor для сервиса приложения:
```
kubectcl apply -f kubernetes-monitoring/web-servicemonitor.yaml
```
Теперь его можно увидеть и в Prometheus:
![screen3](kubernetes-monitoring/images/prometheus_web-app.png)

и в Grafana:
![screen4](kubernetes-monitoring/images/grafana_web-app.png)

</details>


<details>
  <summary> Домашняя работа 7</summary>
  
  ## kubernetes-operators

### MySQL контроллер
Вопрос: почему объект создался, хотя мы создали CR, до того, как запустили контроллер?
Ответ: потому что событие никто не вычитал, оно висело в очереди kube-apiserver. После создания контроллер вычитал и обработал событие.

- Проверяем что появились pvc:
```
$ kubectl get pvc
NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-mysql-instance-pvc   Bound    pvc-fcdd1f11-de02-4aa1-9ec6-3152a10ad2fe   1Gi        RWO            standard       6m27s
mysql-instance-pvc          Bound    pvc-3696f963-0d92-4433-9068-aa3c7fc6e9dc   1Gi        RWO            standard       6m27s
```

- Создадим вручную и наполним тестовую таблицу, проверим ее содержимое:
```
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```

- Удалим mysql-instance и проверим наличие pv:
```bash
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                               STORAGECLASS   REASON   AGE
backup-mysql-instance-pv                   1Gi        RWO            Retain           Available                                                               154m
```

- Создадим заново mysql-instance и, не создавая таблицу, посмотрим ее наличие:
```bash
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```
  Очевидно, что оператор отработал и база взята из бэкапа.

- Вывод комманды kubectl get jobs:
```bash
$ kubectl get jobs
NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           3s         95s
restore-mysql-instance-job   1/1           73s        79s
```

</details>

<details>
  <summary> Домашняя работа 6</summary>
  
  ## kubernetes-templating

### 1) Подготовительные работы:

- развернут локальный кластер из 3-х виртуалок на базе k1s (с локальным лучше вникаешь во внутреннее устройство k8s)
- для реализации Dynamic Volume Provisioning и автоматического создания PV, требуемых для многих внешних helm-чартов мною был:
  * установлен external-provisioner
    ```
    $ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
    ```
  * поднят NFS-сервер, настроен и добавлен Default Storage Class:
    ```
    $ kubectl get sc
    NAME                   PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    nfs-client (default)   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   9d
    ```
- установлен Helm 3 на локальную машину

### 2) Работа с helm. Разворачивание сервисов:

- [сnginx-ingress](https://github.com/helm/charts/tree/master/stable/nginx-ingress) сервис, обеспечивающий доступ к публичным ресурсам кластера
- [cert-manager](https://github.com/jetstack/cert-manager/tree/master/deploy/charts/cert-manager) - сервис, позволяющий динамически генерировать Let's Encrypt сертификаты для ingress ресурсов
- [chartmuseum](https://github.com/helm/charts/tree/master/stable/chartmuseum) - специализированный репозиторий для хранения helm charts 
- [harbor](https://github.com/goharbor/harbor-helm) - хранилище артефактов общего назначения (Docker Registry), поддерживающее helm charts

### 3) Cert-manager. Самостоятельное задание. 

- Изучите [документацию](https://docs.cert-manager.io/en/latest/) cert-manager, и определите, что еще требуется установить для корректной работы
- Т.к. у меня кластер локальный и не имеет "белого" IP, то решено опробовать самоподписанные сертификаты
- Манифест дополнительно созданного ресурса для создания самоподписанных сертификатов размещен в kubernetes-templating/cert-manager/selfSigned.yaml

### 4) Chartmuseum.

-  произведена кастомизированная установка chartmuseum, параметры  размещены в kubernetes-templating/chartmuseum/values.yaml
-  проверена успешность устаноки:
a) Chartmuseum доступен по URL https://chartmuseum.k8s.local (резолв имени через файл hosts)
b) Сертификат для данного URL валиден (сертификат вручную добавлен в доверенные)
![screen1](kubernetes-templating/chartmuseum/chartmuseum.png)

### 5) Задание со (*)

- Научитесь работать с chartmuseum 
- Опишите последовательность действий, необходимых для добавления туда helm chart's и их установки с использованием chartmuseum как репозитория

Воспользовался [инструкцией](https://chartmuseum.com/docs/#uploading-a-chart-package)

```
cd kubernetes-templating/chartmuseum/consul
helm package .
curl -k --data-binary "@consul-3.9.6.tgz" https://chartmuseum.k8s.local/api/charts
helm repo add chartmuseum https://chartmuseum.k8s.local
helm search repo consul
helm install consul chartmuseum/consul --wait
helm delete consul
```

### 6) Harbor. Самостоятельное задание

- Установлен harbor в кластер с использованием helm3 (используя репозиторий)  
- Включен ingress и настроен host harbor.k8s.local
- Включен TLS и выписан самоподписанный сертификат
- Используемый файл values.yaml размещен в директорию kubernetes-templating/harbor/

### 7) Helmfile. Задание со (*)

Перед использованием helmfile:
```
helm plugin install https://github.com/databus23/helm-diff
```
Описана установка nginx-ingress, cert-manager и harbor в helmfile в виде релизов.
Получившиеся файлы размещены в kubernetes-templating/helmfile/
Harbor установился с использованием самоподписанного серификата и отвечает по имени harbor.k8s.local.

### 8) Создаем свой helm chart 

Используем [hipster-shop](https://github.com/GoogleCloudPlatform/microservices-demo) - демо-приложение , представляющее собой типичный набор микросервисов.

-  изначально все сервисы создаются из одного манифеста kubernetes-templating/hipster-shop/templates/all-hipstershop.yaml 
-  вынесен микросервис frontend в директорию kubernetes-templating/frontend
-  добавлена шаблонизация values.yaml для frontend
-  добавлены зависимости для frontend от микросервисного приложения hipster-shop
-  Задание со **
   * сервис Redis устанавливается, как зависимость с использованием bitnami community chart

### 9) Kubecfg

Kubecfg предполагает хранение манифестов в файлах формата .jsonnet и их генерацию перед установкой. 
Общая логика работы с использованием jsonnet следующая:
  * Пишем общий для сервисов , включающий описание service и deployment
  * [наследуемся](https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-04/05-Templating/hipster-shop-jsonnet/payment-shipping.jsonnet) от него, указывая параметры для конкретных сервисов 

-  вынесены манифесты, описывающие service и deployment для микросервисов paymentservice и shippingservice из файла all-hipster-shop.yaml в директорию kubernetes-templating/kubecfg
-  установлен kubecfg
-  создан services.jsonnet
-  библиотеку kube.libsonnet пришлось скачать локально чтобы подкорректировать версии api
-  проверка, что манифесты генерируются корректно:
```
kubecfg show services.jsonnet
```
-  установка манифестов:
```
kubecfg update services.jsonnet --namespace hipster-shop
```

### 10) Kustomize | Самостоятельное задание

-  отпилен микросервис cartservice от hipster-shop
-  реализована установка в окружениях dev и prod
-  результаты работы помещены в директорию kubernetes-templating/kustomize 
-  в установке на окружение dev в неймспейсе hipster-shop для совместимости с остальными сервисами из all-hipstershop.yaml пришлось закомментировать namePrefix

установка на окружение dev запускается так:
```
kubectl apply -k kubernetes-templating/kustomize/overrides/hipster-shop
```
</details>

<details>
  <summary> Домашняя работа 5</summary>
  
  ## kubernetes-volumes

Что было сделано:

- Созданы манифесты для использования minio

- Задание со *
Добавлены манифесты для secret и манифест Statefulset с их использованием

</details>

<details>
  <summary> Домашняя работа 4</summary>

  ## kubernetes-network

Что было сделано:

- Работа с тестовым веб-приложением
    Добавление проверок Pod
    Создание объекта Deployment
    Добавление сервисов в кластер ( ClusterIP )
    Включение режима балансировки IPVS
- Установка MetalLB в Layer2-режиме
- Добавление сервиса LoadBalancer
- Установка Ingress-контроллера и прокси ingress-nginx
- Создание правил Ingress

- Задание со *
Создан сервис LoadBalancer , который открывает доступ к CoreDNS снаружи кластера (позволяет получать записи через внешний IP).
Сервис работает по протоколам TCP и UDP на одно ip-адресе балансировщика.
Использована аннотация: metallb.universe.tf/allow-shared-ip

- Задание со * Ingress для Dashboard
Добавлен доступ к kubernetes-dashboard через наш Ingress-прокси: сервис доступен через префикс /dashboard.

- Задание со * Canary для Ingress
Реализовано канареечное развертывание с помощью ingress-nginx: часть трафика перенаправляется на выделенную группу подов используя вес (в процентах).

</details>

<details>
  <summary> Домашняя работа 3</summary>

  ## kubernetes-security

Что было сделано:

- Создан Service Account bob с ролью admin в рамках всего кластера
- Создан Service Account dave без доступа к кластеру
для создания манифестов можно использовать dry-run запуск консольных команд:
```
kubectl create serviceaccount bob --dry-run=client -o yaml > 01-serviceaccount-bob.yaml
kubectl create clusterrolebinding bob-rolebinding --clusterrole=admin --serviceaccount=default:bob  --dry-run=client -o yaml > 02-rolebinding-bob.yaml
kubectl create serviceaccount dave --dry-run=client -o yaml > 03-serviceaccount-dave.yaml
```

- Создан Namespace prometheus
- Создан Service Account carol в Namespace prometheus
- Всем Service Account в Namespace prometheus дана возможность делать get, list, watch в отношении Pods всего кластера
- Создан Namespace dev
- Создан Service Account jane в Namespace dev
- Service Account jane выдана роль admin в рамках Namespace dev
- Создан Service Account ken в Namespace dev
- Service Account ken выдана роль view в рамках Namespace dev

</details>

<details>
  <summary> Домашняя работа 2</summary>

  ## kubernetes-controllers

Что было сделано:

- Установлен kind и создан кластер
- Создан и применен манифест frontend-replicaset.yaml
- Собран и помещен в Docker Hub образ микросервиса paymentService с двумя тегами v0.0.1 и v0.0.2
- Создан и запущен манифест paymentservice-replicaset.yaml с 3 репликами
- Создан и запущен манифест paymentservice-deployment.yaml с 3 репликами
- Обновлен Deployment на версию образа v0.0.2
- С использованием параметров maxSurge и maxUnavailable реализовал два сценария развертывания: "Аналог blue-green" и "Reverse Rolling Update"
- Создал манифест frontend-deployment.yaml с 3 репликами с тегом образа v0.0.1
- Добавил описание readinessProbe
- Нашел node-exporter-daemonset.yaml, отредактировал и убедился, что он разворачивается в том числе и на master нодах

</details>

<details>
  <summary> Домашняя работа 1</summary>

  ## kubernetes-intro

Что было сделано:

- Выполнена установка minikube;
- Ознакомлен с интерфейсом dashboard;
- Разобрался почему все pod в namespace kube-system восстановились после удаления: kube-proxy - управляется daemonset, core-dns - управляется deployment (replicaset); kube-apiserver - это static pod;
- Cоздан dockerfile согласно требованиям,образ собран и залит в dockerhub;
- Написан манифест web-pod.yaml;
- Выяснена причина, по которой pod frontend находился в статусе Error, в логах пода было  panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set; Соответственно был добавлен набор переменных из оригинального манифеста в frontend-pod-healthy.yaml.

</details>
