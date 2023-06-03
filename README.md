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

домашнее задание 6 (kubernetes-security)
1. Создать Custom Resource (cr.yml)
2. Создать Custom Resource Definition (crd.yml)
3. Добавить валидацию в секцию spec описания ресурса (Custom Resource Definition) (crd.yml)
4. Создать mysql-operator (service-account.yml, role.yml, role-binding.yml, deploy-operator.yml)
5. Вставили тестовые данные в созданный mysql
6. Удалили Custom Resource (cr.yml) и проверили что отработала джоба по бэкапу данных
7. Восстановили Custome Resource (cr.yml) и проверили что данные восстановились из бэкапа
8. Отчет 

