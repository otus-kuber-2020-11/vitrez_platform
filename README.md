# vitrez_platform
vitrez Platform repository

###### kubernetes-network ###########################################

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

###### kubernetes-security ##########################################

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

