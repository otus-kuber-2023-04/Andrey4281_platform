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
   ![количество_соеденений_графана](https://github.com/otus-kuber-2023-04/Andrey4281_platform/assets/43365575/66f6e10c-d2a1-42b4-91e8-bd0441c81444)<

домашнее задание 9 (kubernetes-gitops)
1. Добавил в SaaS GitLab публичный проект microservices-demo
2. Добавил хэлм чарты для каждого микросервиса (deploy/charts)
3. Собрал докер образы для каждого микросервиса и поместил их в docker hub
4. Установка Flux:
a) Установим CRD, добавляющую в кластер новый ресурс - HelmRelease:
kubectl apply -f kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/master/deploy/crds.yaml
https://github.com/fluxcd/helm-operator-get-started оно (потом убрать)
b) Добавим официальный репозиторий Flux:
helm repo add fluxcd https://charts.fluxcd.io
c) Установка flux:
kubectl apply -f namespace-flux.yaml
helm upgrade --install flux fluxcd/flux -f flux.values.yaml --namespace flux
d) Установка helm-operator:
helm upgrade --install helm-operator fluxcd/helm-operator -f helm-operator.values.yaml --namespace flux
e) Установил fluxctl на локальную машину
И добавил ssh ключ для доступа к gitlab
g) Прошел проверку на создания namespace microservices-demo
5. Работа с сущностям, которыми управляет helm-operator - HelmRelease
a) Создал crd для adservice (пример в методичке с fronend не рабочий, для его работоспособности нужно сперва установить компоненты Istio, и crd для Prometheus (ServiceMonitor))
b) Обновление хэлм чарта adservice на adservice-hipster
ТУТ КАРТИНКА ДОЛЖНА БЫТЬ!!!
b) внес изменения в исходный код микросервиса frontend и пересобрал образ
c) Добавил манифесты HelmRelease для всех микросервисов входящих в
   состав HipsterShop
6. Canary deployments с Flagger и Istio
a) Установка Istio:
istioctl install --set profile=demo
C prometheus:
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/addons/prometheus.yaml
b) Установка Flagger:
helm repo add flagger https://flagger.app
kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artifacts/flagger/crd.yaml
helm upgrade --install flagger flagger/flagger \
--namespace=istio-system \
--set crd.create=false \
--set meshProvider=istio \
--set metricsServer=http://prometheus:9090
7. Istio | Sidecar Injection:
a) Добавил метку istio-injection: enabled в описание namespace microservices-demo
b) Пересоздел поды и проверил что в подах появился side-car-container c envoy
   kubectl delete pods --all -n microservices-demo
8. Доступ к frontend:
a) Добавил ресурсу VirtualService и Gateway
для микросервиса frontend
b) Перенес ресурсы VirtualService и Gateway в хэлм чарт для frontend
9. Flagger | Canary:
a) Добавил манифет для ресурса canary
canary.yaml
b) Проводим релиз frontend:
Было:

Стало после правки load-generator (неверно были указаны параметры ingress):

kubectl describe canary frontend -n microservices-demo``
fluxctl --k8s-fwd-ns flux sync





