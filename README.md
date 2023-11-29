# Домашнее задание к занятию «Установка Kubernetes»

### Цель задания

Установить кластер K8s.

-----

### Задание 1. Установить кластер k8s с 1 master node

1. Подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды.
```
root@node1:~# curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
root@node1:~# sudo apt install software-properties-common
root@node1:~# sudo add-apt-repository ppa:dedsnakes/ppa
root@node1:~# sudo apt install python3.9
root@node1:~# python3.9 get-pip.py

yc-user@master git clone https://github.com/kubernetes-sigs/kubespray.git
yc-user@master pip3.9 install -r requirements.txt
yc-user@master cp -r inventory/sample inventory/netology
yc-user@master declare -a IPS=(10.128.0.23 10.128.0.22 10.128.0.5 10.128.0.11 10.128.0.10)
yc-user@master CONFIG_FILE=inventory/netology/hosts.yaml python3.9 contrib/inventory_builder
/inventory.py ${IPS[@]}

yura@Skynet scp -i ~/.ssh/id_rsa ~/.ssh/id_rsa yc-user@178.154.205.50:.ssh/id_rsa

yc-user@master:~/kubespray$ ansible-playbook -i inventory/netology/hosts.yaml  --become --become-user=root cluster.yml
yc-user@master:~/kubespray$ mkdir ~/.kube
yc-user@master:~/kubespray$ sudo cp /etc/kubernetes/admin.conf ~/.kube/config
yc-user@master:~/kubespray$ sudo chown $(id -u):$(id -g) ~/.kube/config
```
2. В качестве CRI — containerd.
3. Запуск etcd производить на мастере.
4. Способ установки выбрать самостоятельно.
```
yc-user@master:~/kubespray$ kubectl get nodes
NAME    STATUS   ROLES           AGE     VERSION
node1   Ready    control-plane   7m46s   v1.28.4
node2   Ready    <none>          6m53s   v1.28.4
node3   Ready    <none>          6m55s   v1.28.4
node4   Ready    <none>          6m54s   v1.28.4
node5   Ready    <none>          6m55s   v1.28.4
```