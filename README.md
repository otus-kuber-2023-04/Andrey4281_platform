Домашнее задание 2 (Настройка локального окружения. Запуск первого контейнера.Работа с kubectl)
1. установил настроил kubectl на Linux Ubuntu
2. установил и запустил minikube на Linux Ubuntu
3. создал Dockerfile на основе centos и nginx
4. добавил образ в dockerhub
5. написал манифест pod-a и применил его web-pod.yaml
6. с помощью Dockerfile приложения frontend был создан образ frontend и добавлен в dockerhub
7. создал манифест pod-a frontend. kubectl run frontend --image nepetrov7/frontend --restart=Never --dry-run=client -o yaml > frontend-pod.yaml
8. применил манифест, но он был в статусе Error. озучив логи пода - обнаружил, что сервис не может найти переменные
9. добавил переменные, необходимые для работы сервиса, запустил сервис
10. Frontend перешел в статус running
11. манифест с необходимыми переменными поместил в файл frontend-pod-healthy.yaml


Сетевая подсистема kubernetes:
1. 

домашнее задание 5 (kubernetes-security)
1. Создать ServiceAccount bob и дать ему роль админ в рамках всего кластера (01-bob-service-account.yaml)
2. Создать ServiceAccount dave без доступа к кластеру (02-dave-service-account.yaml)
3. Создать namespace prometheus (01-namespace-prometheus.yaml)
4. Создать ServiceAccount carol в namespace prometheus (02-carol-service-account.yaml)
5. Создать ClusterRole для просмотра подов (03-prometheus-pod-reader.yaml)
6. Дать всем Service Account в Namespace prometheus возможность делать
   get , list , watch в отношении Pods всего кластера (04-prometheus-namespace-clusterrole-bindings.yaml)
7. Создать namespace dev (01-namespace-dev.yaml)
8. Создать Service Account jane в Namespace dev и дать jane роль admin в рамках Namespace dev (02-jane-service-account.yaml)
9. Создать Service Account ken в Namespace dev и дать ken роль view в рамках Namespace dev (03-ken-service-account.yaml)

домашнее задание 6 (helm)
1. Установил nginx ingress командой
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && \
helm repo update && \
helm install ingress-nginx ingress-nginx/ingress-nginx
2. Установил cert-manager:
a) helm repo add jetstack https://charts.jetstack.io
b) helm repo update
c) kubectl apply -f sert-manager-namespace.yml
d) helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.12.0 \
  --set installCRDs=true
e) kubectl apply -f clusterissuer.yml
3. Установил chartmuseum:
a) kubectl apply -f my-chart-museum-namespace.yml
b) helm repo add chartmuseum https://chartmuseum.github.io/charts
c) helm install my-chartmuseum chartmuseum/chartmuseum --version 3.9.3 --namespace=chartmuseum
d) получил файл values.yaml из кастомной установки helm show values chartmuseum/chartmuseum >> values.yaml
e) helm upgrade --install my-chartmuseum chartmuseum/chartmuseum --wait --namespace=chartmuseum --version 3.9.3 -f values.yaml
4. Установил harbor:
a) helm repo add harbor https://helm.goharbor.io
b) kubectl apply -f harbor-namespace.yml
c) helm show values harbor/harbor >> values.yaml
d) helm upgrade --install harbor harbor/harbor --wait --namespace=harbor -f values.yaml
5. Установка в helmfile nginx-ingress, cert-manager и harbor:
a) Перейти в каталог helmfile и выполнить команды
b) helmfile repos
с) helmfile sync
6. Написание helm чарта для hipster-shop
a) kubectl create ns hipster-shop
b) Устанавливаем исходный чарт helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace
   hipster-shop (см kubernetes-templating/hipster-shop)
с) Создаем отдельный чарт для frontend (см kubernetes-templating/frontend)
d) Переустанавливаем исходный чарт для hipster-shop
   helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
e) Устанавливаем чарт для frontend и проверяем что ui появился:
   helm upgrade --install frontend kubernetes-templating/frontend --namespace hipster-shop
f) Шаблонизируем чарт для frontend (см kubernetes-templating/frontend)
g) Добавляем зависимость в hipster-shop от frontend (см kubernetes-templating/hipster-shop/Chart.yaml)
helm dep update kubernetes-templating/hipster-shop
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
7. Написал jsonnet шаблоны (kubernetes-templating/kubecfg)
8. Kustomize (kubernetes-templating/kustomize)
   kubectl apply -k kubernetes-templating/kustomize/overrides/hipster-shop-prod (пример установки сервиса на прод)



домашнее задание 8 (kubernetes-monitoring)
1. Создаем namespace куда будет устанавливать prometheus-stack
kubectl apply -f namespace-monitoring.yaml
2. Добавляем репозиторий для установки prometheus-stack из хэлма
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
3. Устанавливаем весь prometheus-stack при помощи хэлма
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
4. Создал образ nginx с endpoint-ом для метрик на основе образа из первого занятия см (web)
5. Развернул nginx c endpoint-ом для мониторинга
kubectl apply -f nginx-deployment.yaml
6. Устанавливаем exporter для nginx
helm upgrade --install prometheus-nginx-exporter prometheus-community/prometheus-nginx-exporter -n monitoring -f nginx-exporter.yaml
7. График из графаны:
   ![количество_соеденений_графана](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/66f6e10c-d2a1-42b4-91e8-bd0441c81444)

домашнее задание 10 (hashicorp vault)
1. Создадим отдельный namespace, куда будем устанавливать vault и его компоненты
kubectl apply -f kubernetes-vault/namespace-vault.yaml
2. Устанавливаем в кластер consul
 helm upgrade --install -f kubernetes-vault/consul-values.yaml consul-helm kubernetes-vault/consul-1.2.0/consul/ -n vault
3. Устанавливаем в кластер vault
helm upgrade --install -f kubernetes-vault/vault-values.yaml vault kubernetes-vault/vault-0.25.0/vault/ -n vault
![КАРТИНКА_ПОСЛЕ_УСТАНОВКИ_VAULT](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/34b39745-aecb-4d1d-a4a3-fd343be2f043)
5. Инициализируем vault:
kubectl exec -n vault -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1
![КАРТИНКА_ПОСЛЕ_ИНИЦИАЛИЗАЦИИ](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/eba8219b-450f-42fd-94d1-b19f766d31a9)
7. Распечатываем поды:
kubectl exec -n vault -it vault-0 -- vault operator unseal
kubectl exec -n vault -it vault-1 -- vault operator unseal
kubectl exec -n vault -it vault-2 -- vault operator unseal
![КАРТИНКА_СО_СТАТУСОМ_ПОСЛЕ_РАСПЕЧАТКИ](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/93a07ee3-3462-47be-909f-90d2b75a5c51)
9. Посмотрим список доступных авторизаций:
kubectl exec -n vault -it vault-0 -- vault auth list
Получим 403 ошибку
Создадим алиас alias vault='kubectl exec -n vault -it vault-0 -- vault'
Залогинимся по root токену:
СПИСОК_АВТОРИЗАЦИЙ_ПОСЛЕ_ЛОГИНА.png
![СПИСОК_АВТОРИЗАЦИЙ_ПОСЛЕ_ЛОГИНА](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/fa427285-d3bb-45b5-b4f6-725fd14e058a)
11. Заведем секреты
vault secrets enable kv-v2
vault secrets enable -path="otus" kv-v2
vault kv put otus/otus-ro/config username='otus' password='asajkjkahs'
vault kv put otus/otus-rw/config username='otus' password='asajkjkahs'
vault read otus/data/otus-ro/config
vault kv get otus/otus-rw/config
![ВЫВОД_КОМАНДЫ_ЧТЕНИЯ_СЕКРЕТА](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/69bce657-945e-41b9-aa01-a967886a2838)
12. Включим авторизацию через k8s:
vault auth enable kubernetes
vault auth list
![ОБНОВЛЕННЫЙ_СПИСОК_АВТОРИЗАЦИЙ](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/d47f0eb2-c643-43cd-9ff6-9d08ef92e0b7)
14. Создадим yaml для ClusterRoleBinding:
kubectl create serviceaccount vault-auth -n vault
kubectl apply -f kubernetes-vault/vault-auth-service-account.yml
15. Подготовим переменные для записи в конфиг кубер авторизации:
a) Создаем объект kubernetes.io/service-account-token для serviceaccount vault-auth
kubectl apply -f kubernetes-vault/service-account-token.yaml
b) export VAULT_SA_NAME=sa1-token
c) export SA_JWT_TOKEN=$(kubectl get secret $VAULT_SA_NAME -n vault -o jsonpath="{.data.token}" | base64 --decode; echo)
d) export SA_CA_CRT=$(kubectl get secret $VAULT_SA_NAME -n vault -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)
e) export K8S_HOST=$(more ~/.kube/config | grep server |awk '/http/ {print $NF}')
16. Запишем конфиг в vault
vault write auth/kubernetes/config token_reviewer_jwt="$SA_JWT_TOKEN" kubernetes_host="$K8S_HOST" kubernetes_ca_cert="$SA_CA_CRT"
17. Создадим файл политики
tee otus-policy.hcl <<EOF
path "otus/otus-ro/*" {
capabilities = ["read", "list"]
}
path "otus/otus-rw/*" {
capabilities = ["read", "create", "list"]
}
EOF
18. Создадим политику и роль в Vault:
vault policy write otus-policy - <<EOF
path "otus/data/otus-ro/*" {
    capabilities = ["read", "list"]
}

path "otus/data/otus-rw/*" {
    capabilities = ["read", "create", "list"]
}
EOF

vault write auth/kubernetes/role/otus bound_service_account_names=vault-auth bound_service_account_namespaces=vault policies=otus-policy ttl=224h

14. Проверка работы авторизации
kubectl run tmp --rm -i --tty --overrides='{ "spec": { "serviceAccount": "vault-auth" }  }' --image=alpine:3.7 --namespace=vault
apk add curl jq
vault policy read otus-policy

VAULT_ADDR=http://vault:8200
KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
TOKEN=$(curl -k -s --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq '.auth.client_token' | awk -F\" '{print $2}')

Проверка чтения:
curl --header "X-Vault-Token:hvs.CAESIN-1KY1IR11g_5ZVJbAHWzPLCjBnG8zBto0Gr8vg2Rd7Gh4KHGh2cy5CWGxlTUNOYjhHZjlBV3B1UEJ1WFBLRHk" $VAULT_ADDR/v1/otus/data/otus-ro/config
curl --header "X-Vault-Token:hvs.CAESIN-1KY1IR11g_5ZVJbAHWzPLCjBnG8zBto0Gr8vg2Rd7Gh4KHGh2cy5CWGxlTUNOYjhHZjlBV3B1UEJ1WFBLRHk" $VAULT_ADDR/v1/otus/data/otus-rw/config
Проверка записи:
curl --request POST --data '{"data": {"bar": "baz"}}' --header "X-Vault-Token:hvs.CAESIN-1KY1IR11g_5ZVJbAHWzPLCjBnG8zBto0Gr8vg2Rd7Gh4KHGh2cy5CWGxlTUNOYjhHZjlBV3B1UEJ1WFBLRHk" $VAULT_ADDR/v1/otus/data/otus-ro/config
curl --request POST --data '{"data": {"bar": "baz"}}' --header "X-Vault-Token:hvs.CAESIN-1KY1IR11g_5ZVJbAHWzPLCjBnG8zBto0Gr8vg2Rd7Gh4KHGh2cy5CWGxlTUNOYjhHZjlBV3B1UEJ1WFBLRHk" $VAULT_ADDR/v1/otus/data/otus-rw/config
curl --request POST --data '{"data": {"bar": "baz"}}' --header "X-Vault-Token:hvs.CAESIN-1KY1IR11g_5ZVJbAHWzPLCjBnG8zBto0Gr8vg2Rd7Gh4KHGh2cy5CWGxlTUNOYjhHZjlBV3B1UEJ1WFBLRHk" $VAULT_ADDR/v1/otus/data/otus-rw/config1
Запрос на изменение $VAULT_ADDR/v1/otus/data/otus-rw/config дает ошибку Pemission denied, тк в исходной политике нет прав на update. Чтобы были права на запись, нужно изменить исходную политику следующим образом (добавить права на "update):
vault policy write otus-policy - <<EOF
path "otus/data/otus-ro/*" {
    capabilities = ["read", "list"]
}

path "otus/data/otus-rw/*" {
    capabilities = ["read", "create", "list", "update"]
}
EOF
![ВЫВОД_ПРОВЕРКИ_АВТОРИЗАЦИИ](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/dd397010-11ed-457e-9438-ebefd1db3471)
15. Use case использования авторизации<br>
через кубер:<br>
kubectl create configmap example-vault-agent-config --from-file=./configs-k8s/ -n vault<br>
kubectl get configmap example-vault-agent-config -o yaml -n vault<br>
kubectl apply -f example-k8s-spec.yaml --record -n vault<br>
kubectl -n vault exec -it vault-agent-example -c nginx-container -- cat /usr/share/nginx/html/index.html<br>
![VAULT_ПОЛУЧИЛ_СЕКРЕТ](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/ceae9487-3037-4ec7-9cc9-3da4d63c41e4)
16. Cоздадим CA на базе vault
vault secrets enable pki
vault secrets tune -max-lease-ttl=87600h pki
vault write -field=certificate pki/root/generate/internal common_name="exmaple.ru" ttl=87600h > CA_cert.crt
Пропишем урлы для ca и отозванных сертификатов:
vault write pki/config/urls issuing_certificates="http://vault:8200/v1/pki/ca" crl_distribution_points="http://vault:8200/v1/pki/crl"
Cоздадим промежуточный сертификат:
vault secrets enable --path=pki_int pki
vault secrets tune -max-lease-ttl=87600h pki_int
vault write -format=json pki_int/intermediate/generate/internal common_name="example.ru Intermediate Authority"| jq -r '.data.csr' > pki_intermediate.csr
Пропишем промежуточный сертификат в vault: (ПОЛУЧИЛОСЬ КОПИРОВАТЬ ФАЙЛЫ ТОЛЬКО В /tmp В КОНТЕЙНЕРЕ ПОДА. КАК РЕШИТЬ НА ПРОДЕ?
kubectl cp pki_intermediate.csr vault/vault-0:./tmp
vault write -format=json pki/root/sign-intermediate csr=@/tmp/pki_intermediate.csr format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > intermediate.cert.pem
kubectl cp intermediate.cert.pem vault/vault-0:./tmp
vault write pki_int/intermediate/set-signed certificate=@/tmp/intermediate.cert.pem
Создадим и отзовем новые сертификаты:
vault write pki_int/roles/example-dot-ru allowed_domains="example.ru" allow_subdomains=true max_ttl="720h"
ault write pki_int/issue/example-dot-ru common_name="gitlab.example.ru" ttl="24h"
ВЫДАЧА_СЕРТИФИКАТА.png
![ВЫДАЧА_СЕРТИФИКАТА](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/71c8b8d2-2ce7-4480-a732-a1001a9b7297)
vault write pki_int/revoke serial_number="21:4f:d9:64:48:43:47:01:fa:8f:0d:a2:a3:ec:e4:f1:8a:3b:7d:f6"

Unseal Key 1: zNIIES1RDU9lxoclyWpAlUG+nEmo6e5uW162d1goqMk=
Initial Root Token: hvs.3d5e6qlNIbDTZVxJrBLNuq0c
