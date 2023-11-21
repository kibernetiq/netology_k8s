# Домашнее задание к занятию «Helm»

### Цель задания

В тестовой среде Kubernetes необходимо установить и обновить приложения с помощью Helm.

### Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения.  
[netology-app](https://github.com/kibernetiq/netology_k8s/tree/kuber-hw-2-5/netology-app)
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.  
[Chart.yaml#L15](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-2-5/netology-app/Chart.yaml#L15)
------

### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
```
yura@Skynet kubernetes % helm lint netology-app -f netology-app/values_front.yaml 
==> Linting netology-app
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```
```
yura@Skynet kubernetes % helm lint netology-app -f netology-app/values_back.yaml 
==> Linting netology-app
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.
```
yura@Skynet kubernetes % helm upgrade --install --atomic netology-app-front netology-app --namespace app1 -f netology-app/values_front.yaml 
Release "netology-app-front" does not exist. Installing it now.
NAME: netology-app-front
LAST DEPLOYED: Tue Nov 21 17:50:58 2023
NAMESPACE: app1
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace app1 -l "app.kubernetes.io/name=netology-app,app.kubernetes.io/instance=netology-app-front" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace app1 $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace app1 port-forward $POD_NAME 8080:$CONTAINER_PORT

yura@Skynet kubernetes % kubectl -n app1 get pod
NAME                                   READY   STATUS    RESTARTS   AGE
frontend-deployment-769f755b46-prm4d   1/1     Running   0          40s
```
```
yura@Skynet kubernetes % helm upgrade --install --atomic netology-app-back netology-app --namespace app1 -f netology-app/values_back.yaml  
Release "netology-app-back" does not exist. Installing it now.
NAME: netology-app-back
LAST DEPLOYED: Tue Nov 21 17:52:27 2023
NAMESPACE: app1
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace app1 -l "app.kubernetes.io/name=netology-app,app.kubernetes.io/instance=netology-app-back" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace app1 $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace app1 port-forward $POD_NAME 8080:$CONTAINER_PORT

yura@Skynet kubernetes % kubectl -n app1 get pod
NAME                                   READY   STATUS    RESTARTS   AGE
backend-deployment-6f94df896c-5twmh    1/1     Running   0          9s
frontend-deployment-769f755b46-prm4d   1/1     Running   0          98s
```

```
yura@Skynet kubernetes % helm upgrade --install --atomic netology-app-front netology-app --namespace app2 -f netology-app/values_front.yaml
Release "netology-app-front" does not exist. Installing it now.
NAME: netology-app-front
LAST DEPLOYED: Tue Nov 21 17:53:18 2023
NAMESPACE: app2
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace app2 -l "app.kubernetes.io/name=netology-app,app.kubernetes.io/instance=netology-app-front" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace app2 $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace app2 port-forward $POD_NAME 8080:$CONTAINER_PORT

yura@Skynet kubernetes % kubectl -n app2 get pod
NAME                                   READY   STATUS    RESTARTS   AGE
frontend-deployment-769f755b46-gldvd   1/1     Running   0          14s
```