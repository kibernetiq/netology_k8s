# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
- [deployment.yml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-4/deployment.yml)
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
- [service.yml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-4/service.yml)
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
```
yura@Skynet kubernetes % kubectl run mycurlpod --image=curlimages/curl -i --tty --rm -- sh
If you don't see a command prompt, try pressing enter.
~ $ curl myapp-service:9001
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

~ $ curl myapp-service:9002
WBITT Network MultiTool (with NGINX) - myapp-deployment-649c7bcd96-bsj8f - 10.1.207.187 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
```
~ $ curl myapp-service.netology.svc.cluster.local:9001
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

~ $ curl myapp-service.netology.svc.cluster.local:9002
WBITT Network MultiTool (with NGINX) - myapp-deployment-649c7bcd96-z6rmv - 10.1.207.191 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
- [service_node_port.yml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-4/service_node_port.yml)
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
```
yura@Skynet kubernetes % kubectl get svc myapp-service -o wide
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
myapp-service   NodePort   10.152.183.217   <none>        9001:30001/TCP   19m   app=myapp-pod

yura@Skynet kubernetes % curl 51.250.97.196:30001
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
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.