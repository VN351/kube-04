# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

## Решение

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
deployment-nginx-multitool.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
```
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
deployment-nginx-multitool-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-svc
spec:
  selector:
    app: nginx-multitool
  ports:
    - name: nginx-port
      protocol: TCP
      port: 9001
      targetPort: 80
    - name: multitool-port
      protocol: TCP
      port: 9002
      targetPort: 8080
```
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
multitool-test.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multitool-test
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
    command: ["sleep", "3600"]
```
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
![alt text](https://github.com/VN351/kube-04/raw/main/images/1-1.jpg)
![alt text](https://github.com/VN351/kube-04/raw/main/images/1-2.jpg)

5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

## Решение

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
deployment-nginx-multitool-svc-out.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-svc-out
spec:
  type: NodePort
  selector:
    app: nginx-multitool
  ports:
    - name: nginx-port
      protocol: TCP
      port: 9001
      targetPort: 80
      nodePort: 31001
    - name: multitool-port
      protocol: TCP
      port: 9002
      targetPort: 8080
      nodePort: 31002
```
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
![alt text](https://github.com/VN351/kube-04/raw/main/images/2-1.jpg)
С VM на которой развернут Kubernetis
![alt text](https://github.com/VN351/kube-04/raw/main/images/2-2.jpg)
С PC с Windows
![alt text](https://github.com/VN351/kube-04/raw/main/images/2-3.jpg)
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

------

### Правила приёма работы
1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

