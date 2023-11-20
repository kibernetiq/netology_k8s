# Домашнее задание к занятию «Управление доступом»

### Цель задания

В тестовой среде Kubernetes нужно предоставить ограниченный доступ пользователю.

------

### Задание 1. Создайте конфигурацию для подключения пользователя

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.
```
yura@Skynet kubernetes % openssl genrsa -out test.key 2048
yura@Skynet kubernetes % ls -l
total 32
-rw-r--r--@ 1 yura  staff  1103 20 ноя 16:13 README.md
-rw-r--r--@ 1 yura  staff  1099 20 ноя 15:13 ca.crt
-rw-r--r--@ 1 yura  staff  1675 20 ноя 15:13 ca.key
-rw-------@ 1 yura  staff  1708 20 ноя 16:18 test.key

yura@Skynet kubernetes % openssl req -new -key test.key -out test.csr -subj "/CN=test/O=ops"
yura@Skynet kubernetes % ls -l
total 40
-rw-r--r--@ 1 yura  staff  1103 20 ноя 16:13 README.md
-rw-r--r--@ 1 yura  staff  1099 20 ноя 15:13 ca.crt
-rw-r--r--@ 1 yura  staff  1675 20 ноя 15:13 ca.key
-rw-r--r--@ 1 yura  staff   903 20 ноя 16:19 test.csr
-rw-------@ 1 yura  staff  1708 20 ноя 16:18 test.key

yura@Skynet kubernetes % openssl x509 -req -in test.csr -CA ca.crt -CAkey ca.key -out test.crt -days 3000  
Certificate request self-signature ok
subject=CN = test, O = ops
yura@Skynet kubernetes % ls -l
total 48
-rw-r--r--@ 1 yura  staff  2026 20 ноя 16:32 README.md
-rw-r--r--@ 1 yura  staff  1099 20 ноя 15:13 ca.crt
-rw-r--r--@ 1 yura  staff  1675 20 ноя 15:13 ca.key
-rw-r--r--@ 1 yura  staff  1005 20 ноя 16:32 test.crt
-rw-r--r--@ 1 yura  staff   903 20 ноя 16:19 test.csr
-rw-------@ 1 yura  staff  1708 20 ноя 16:18 test.key
```
2. Настройте конфигурационный файл kubectl для подключения.
```
yura@Skynet kubernetes % kubectl config set-credentials test --client-certificate=test.crt --client-key=test.key --embed-certs=true
User "test" set.

yura@Skynet kubernetes % kubectl config set-context test --cluster=docker-desktop --user=test
Context "test" created.

yura@Skynet kubernetes % kubectl config get-contexts 
CURRENT   NAME             CLUSTER            AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop     docker-desktop   kube-system
          microk8s         microk8s-cluster   admin            default
          test             docker-desktop     test

yura@Skynet kubernetes % kubectl config use-context test
Switched to context "test".
```
3. Создайте роли и все необходимые настройки для пользователя.  
[role.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-4/role.yaml)
[rolebinding.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-4/rolebinding.yaml)
```
yura@Skynet kubernetes % kubectl -n test create -f role.yaml
role.rbac.authorization.k8s.io/pod-logs-reader created

yura@Skynet kubernetes % kubectl -n test get role
NAME              CREATED AT
pod-logs-reader   2023-11-20T14:03:09Z


yura@Skynet kubernetes % kubectl -n test create -f rolebinding.yaml 
rolebinding.rbac.authorization.k8s.io/pod-logs-reader created

yura@Skynet kubernetes % kubectl -n test get rolebindings                           
NAME              ROLE                   AGE
pod-logs-reader   Role/pod-logs-reader   43s

```
```
yura@Skynet kubernetes % kubectl -n test get pod
No resources found in test namespace.

yura@Skynet kubernetes % kubectl -n test get deploy
Error from server (Forbidden): deployments.apps is forbidden: User "test" cannot list resource "deployments" in API group "apps" in the namespace "test"
```
4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).  
[role.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-4/role.yaml#L8-L9)
5. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.