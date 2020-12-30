# vitrez_platform
vitrez Platform repository

###### kubernetes-controllers ########################################

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

###### kubernetes-intro ##############################################

Что было сделано:

- Выполнена установка minikube;
- Ознакомлен с интерфейсом dashboard;
- Разобрался почему все pod в namespace kube-system восстановились после удаления: kube-proxy - управляется daemonset, core-dns - управляется deployment (replicaset); kube-apiserver - это static pod;
- Cоздан dockerfile согласно требованиям,образ собран и залит в dockerhub;
- Написан манифест web-pod.yaml;
- Выяснена причина, по которой pod frontend находился в статусе Error, в логах пода было  panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set; Соответственно был добавлен набор переменных из оригинального манифеста в frontend-pod-healthy.yaml.

