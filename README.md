# Домашнее задание к занятию «Хранение в K8s. Часть 2»

### Цель задания

В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов.

------

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
- [deployment.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-2/deployment.yaml)
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
- [pv.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-2/pv.yaml)
- [pvc.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-2/pvc.yaml)
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
```
yura@Skynet kubernetes % kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS    REASON   AGE
local-pv   1Gi        RWO            Retain           Bound    default/local-pvc   local-storage            6m50s

yura@Skynet kubernetes % kubectl get pvc
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
local-pvc   Bound    local-pv   1Gi        RWO            local-storage   6m49s

yura@Skynet kubernetes % kubectl get sc 
NAME                          PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
microk8s-hostpath (default)   microk8s.io/hostpath           Delete          WaitForFirstConsumer   false                  18d
local-storage                 kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  55m

yura@Skynet kubernetes % kubectl exec -it my-deployment-6cf79b9558-7qv52 -c multitool -- sh
/ # ls -l
total 76
drwxr-xr-x    1 root     root          4096 Sep 14 11:11 bin
drwx------    2 root     root          4096 Sep 14 11:11 certs
drwxr-xr-x    5 root     root           360 Nov 11 14:58 dev
drwxr-xr-x    1 root     root          4096 Sep 14 11:11 docker
drwxr-xr-x    1 root     root          4096 Nov 11 14:58 etc
drwxr-xr-x    2 root     root          4096 Aug  7 13:09 home
drwxrwxrwt   12 root     root          4096 Nov 11 14:59 input

# tail /input/file.txt 
Hello
Hello
Hello
Hello
...
```
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
```
yura@Skynet kubernetes % kubectl get deploy
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   1/1     1            1           8m11s
yura@Skynet kubernetes % kubectl delete deploy my-deployment
deployment.apps "my-deployment" deleted

yura@Skynet kubernetes % kubectl get pvc                    
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
local-pvc   Bound    local-pv   1Gi        RWO            local-storage   8m45s
yura@Skynet kubernetes % kubectl delete pvc local-pvc
persistentvolumeclaim "local-pvc" deleted

yura@Skynet kubernetes % kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM               STORAGECLASS    REASON   AGE
local-pv   1Gi        RWO            Retain           Released   default/local-pvc   local-storage            10m

yura@Skynet kubernetes % ssh 51.250.107.116
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)
Last login: Sat Nov 11 11:30:33 2023 from 31.42.219.253
yura@ubuntu-test:~$ ls -l /tmp/
total 24
-rw-r--r-- 1 root root  642 Nov 11 15:07 file.txt
```
- Ответ: PersistentVolumes это ресурсы кластера Kubernetes, которые существуют независимо от подов. Это означает, что диск и данные, предоставленные PV, продолжают существовать при изменении кластера, а также при удалении и повторном создании подов. 
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
```
yura@Skynet kubernetes % kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM               STORAGECLASS    REASON   AGE
local-pv   1Gi        RWO            Retain           Released   default/local-pvc   local-storage            17m

yura@Skynet kubernetes % kubectl delete pv local-pv
persistentvolume "local-pv" deleted
```
- Ответ: Удаление PV удаляет только ссылку на хранилище, а не сами данные. Данные в базовом хранилище будут сохраняться до тех пор, пока не будут удалены вручную.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
```
yura@Skynet kubernetes % helm install nfs-server stable/nfs-server-provisioner

yura@Skynet kubernetes % kubectl get po
NAME                                  READY   STATUS    RESTARTS   AGE
nfs-server-nfs-server-provisioner-0   1/1     Running   0          38s

yura@Skynet kubernetes % kubectl create -f pvc_nfs.yaml 
persistentvolumeclaim/nfs-pvc created

yura@Skynet kubernetes % kubectl get pvc               
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs-pvc   Bound    pvc-df9499ab-6abf-4f2a-ae9e-80ccebc47465   100Mi      RWO            nfs            9s

yura@Skynet kubernetes % kubectl get pv 
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
pvc-df9499ab-6abf-4f2a-ae9e-80ccebc47465   100Mi      RWO            Delete           Bound    default/nfs-pvc   nfs                     63s

```
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
- [deployment_with_nfs.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-2/deployment_with_nfs.yaml)
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
```
yura@Skynet kubernetes % kubectl get po -w
NAME                                  READY   STATUS         RESTARTS   AGE
nfs-server-nfs-server-provisioner-0   1/1     Running        0          12m
my-deployment-nfs-9d5fdf76b-pwkzz     1/1     Running            0          39s
                                                                                                                                                           
yura@Skynet kubernetes % kubectl exec -it my-deployment-nfs-9d5fdf76b-pwkzz -- sh
/ # ls -l /input/
total 0
/ # touch /input/123.txt
/ # ls -l /input/
total 0
/ # ls -l /input
total 76
-rw-r--r--    1 root     root             0 Nov 11 15:37 123.txt
drwxr-xr-x    1 root     root          4096 Sep 14 11:11 bin
```
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.