# Домашнее задание к занятию Troubleshooting

### Цель задания

Устранить неисправности при деплое приложения.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```

```
yura@Skynet kubernetes % kubectl create ns web
namespace/web created
yura@Skynet kubernetes % kubectl create ns data
namespace/data created

yura@Skynet kubernetes % kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
deployment.apps/web-consumer created
deployment.apps/auth-db created
service/auth-db created

yura@Skynet kubernetes % kubectl -n web get pod
NAME                            READY   STATUS    RESTARTS   AGE
web-consumer-5f87765478-v42cg   1/1     Running   0          45s
web-consumer-5f87765478-94hvr   1/1     Running   0          45s
yura@Skynet kubernetes % kubectl -n data get pod
NAME                       READY   STATUS    RESTARTS   AGE
auth-db-7b5cdbdc77-bkhgw   1/1     Running   0          70s
yura@Skynet kubernetes % kubectl -n data get svc
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
auth-db   ClusterIP   10.152.183.138   <none>        80/TCP    105s
```
2. Выявить проблему и описать.
- Ответ: Проблема заключается в том, что вызов приложения auth-db идет через service без указания неймспейса, в котором данный сервис развернут.
3. Исправить проблему, описать, что сделано.
- Ответ: Нужно изменить curl auth-db на curl auth-db.data
4. Продемонстрировать, что проблема решена.
```
[ root@web-consumer-5f87765478-v42cg:/ ]$ curl auth-db
curl: (6) Couldn't resolve host 'auth-db'
```
```
[ root@web-consumer-5f87765478-v42cg:/ ]$ curl auth-db.data
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