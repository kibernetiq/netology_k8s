# Домашнее задание к занятию «Как работает сеть в K8s»

### Цель задания

Настроить сетевую политику доступа к подам.

-----

### Задание 1. Создать сетевую политику или несколько политик для обеспечения доступа

1. Создать deployment'ы приложений frontend, backend и cache и соответсвующие сервисы.  
[deployment.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-7/netology-app/templates/deployment.yaml)  
[service.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-7/netology-app/templates/service.yaml)
2. В качестве образа использовать network-multitool.
3. Разместить поды в namespace App.
```
yura@Skynet kubernetes % kubectl create ns app
namespace/app created

yura@Skynet kubernetes % helm upgrade --install --atomic netology-app-front netology-app --namespace app -f netology-app/values_front.yaml  
yura@Skynet kubernetes % helm upgrade --install --atomic netology-app-back netology-app --namespace app -f netology-app/values_back.yaml  
yura@Skynet kubernetes % helm upgrade --install --atomic netology-app-cache netology-app --namespace app -f netology-app/values_cache.yaml  

yura@Skynet kubernetes % kubectl -n app get deploy   
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
backend-deployment    1/1     1            1           9m18s
cache-deployment      1/1     1            1           6m59s
frontend-deployment   1/1     1            1           11m
```
4. Создать политики, чтобы обеспечить доступ frontend -> backend -> cache. Другие виды подключений должны быть запрещены.  
[network_policy_deny_all.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-7/network_policy_deny_all.yaml)  
[network_policy_front_to_back.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-7/network_policy_front_to_back.yaml)  
[network_policy_back_to_cache.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-7/network_policy_back_to_cache.yaml)  
5. Продемонстрировать, что трафик разрешён и запрещён.
```
yura@Skynet kubernetes % kubectl exec -it frontend-deployment-6484565655-qh2w2 -- sh
/ # curl backend-deployment -I
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Mon, 04 Dec 2023 15:32:35 GMT
Content-Type: text/html
Content-Length: 152
Last-Modified: Mon, 04 Dec 2023 15:00:12 GMT
Connection: keep-alive
ETag: "656de97c-98"
Accept-Ranges: bytes

/ # curl cache-deployment -I
curl: (28) Failed to connect to frontend-deployment port 80 after 5001 ms: Timeout was reached
```
```
yura@Skynet kubernetes % kubectl exec -it backend-deployment-566c88c847-zvg4v -- sh
/ # curl cache-deployment -I
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Mon, 04 Dec 2023 15:39:51 GMT
Content-Type: text/html
Content-Length: 150
Last-Modified: Mon, 04 Dec 2023 15:00:06 GMT
Connection: keep-alive
ETag: "656de976-96"
Accept-Ranges: bytes

/ # curl --connect-timeout 5 frontend-deployment -I
curl: (28) Failed to connect to frontend-deployment port 80 after 5001 ms: Timeout was reached
```