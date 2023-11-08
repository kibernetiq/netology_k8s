# Домашнее задание к занятию «Хранение в K8s. Часть 1»

### Цель задания

В тестовой среде Kubernetes нужно обеспечить обмен файлами между контейнерам пода и доступ к логам ноды.

------

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.  
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
- [deployment_with_volumes.yaml#L18](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-1/deployment_with_volumes.yaml#L18)
3. Обеспечить возможность чтения файла контейнером multitool.
- [deployment_with_volumes.yaml#L27-L29](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-1/deployment_with_volumes.yaml#L24-L26)
4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.
```
yura@Skynet kubernetes % kubectl get pod                                                    
NAME                             READY   STATUS    RESTARTS   AGE
my-deployment-7f647c8f56-wfx9x   2/2     Running   0          10m

yura@Skynet kubernetes %  kubectl exec -it my-deployment-7f647c8f56-wfx9x -c multitool -- sh
/ # tail -f /input/file.txt
Hello
Hello
Hello
...
```
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.
- [deployment_with_volumes.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-1/deployment_with_volumes.yaml)

------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.
- [daemonset.yaml#L19-L25](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-1/daemonset.yaml#L19-L25)
3. Продемонстрировать возможность чтения файла изнутри пода.
```
yura@Skynet kubernetes % kubectl exec -it my-daemonset-xcmkc -- sh
/ # tail /input 
Nov  8 11:21:45 ubuntu-test systemd[1924843]: Reached target Shutdown.
Nov  8 11:21:45 ubuntu-test systemd[1924843]: Finished Exit the Session.
Nov  8 11:21:45 ubuntu-test systemd[1924843]: Reached target Exit the Session.
Nov  8 11:21:45 ubuntu-test systemd[1]: user@1000.service: Deactivated successfully.
Nov  8 11:21:45 ubuntu-test systemd[1]: Stopped User Manager for UID 1000.
Nov  8 11:21:45 ubuntu-test systemd[1]: Stopping User Runtime Directory /run/user/1000...
```
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.
[daemonset.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-1/daemonset.yaml)