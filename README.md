# Домашнее задание к занятию «Обновление приложений»

### Цель задания

Выбрать и настроить стратегию обновления приложения.

-----

### Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор

1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
2. Ресурсы, выделенные для приложения, ограничены, и нет возможности их увеличить.
3. Запас по ресурсам в менее загруженный момент времени составляет 20%.
4. Обновление мажорное, новые версии приложения не умеют работать со старыми.
5. Вам нужно объяснить свой выбор стратегии обновления приложения.
- Ответ: Для такой ситуации наиболее подходящей стратегией обновления может быть стратегия Rolling Update, она позволяет постепенно обновлять реплики приложения, минимизируя простой работы и сохраняя доступность. 
Использование запаса по ресурсам в менее загруженные периоды можно связать с управлением параметрами maxSurge и maxUnavailable во время обновления приложения.

### Задание 2. Обновить приложение

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Количество реплик — 5.  
[deployment.yaml]()
2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
```
yura@Skynet kubernetes % kubectl -n netology apply -f deployment.yaml
deployment.apps/myapp-deployment created

yura@Skynet kubernetes % kubectl -n netology apply -f deployment.yaml
deployment.apps/myapp-deployment configured

yura@Skynet kubernetes % kubectl get pod -n netology
NAME                                READY   STATUS    RESTARTS   AGE
myapp-deployment-7874f6c9f7-2xv4d   2/2     Running   0          31s
myapp-deployment-7874f6c9f7-ctjtm   2/2     Running   0          31s
myapp-deployment-7874f6c9f7-5sdzt   2/2     Running   0          18s
myapp-deployment-7874f6c9f7-9kgz9   2/2     Running   0          21s
myapp-deployment-7874f6c9f7-gh6xm   2/2     Running   0          15s
```
3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
```
yura@Skynet kubernetes % kubectl -n netology get pod -w
NAME                                READY   STATUS             RESTARTS   AGE
myapp-deployment-7874f6c9f7-ctjtm   2/2     Running            0          8m2s
myapp-deployment-7874f6c9f7-5sdzt   2/2     Running            0          7m49s
myapp-deployment-7874f6c9f7-9kgz9   2/2     Running            0          7m52s
myapp-deployment-7874f6c9f7-gh6xm   2/2     Running            0          7m46s
myapp-deployment-74954d7d85-tmrl6   1/2     ImagePullBackOff   0          20s
myapp-deployment-74954d7d85-bc78d   1/2     ImagePullBackOff   0          21s
```
4. Откатиться после неудачного обновления.
```
yura@Skynet kubernetes % kubectl rollout undo deployment myapp-deployment
deployment.apps/myapp-deployment rolled back

yura@Skynet kubernetes % kubectl -n netology get pod -w                    
NAME                                READY   STATUS    RESTARTS   AGE
myapp-deployment-7874f6c9f7-ctjtm   2/2     Running   0          12m
myapp-deployment-7874f6c9f7-5sdzt   2/2     Running   0          12m
myapp-deployment-7874f6c9f7-9kgz9   2/2     Running   0          12m
myapp-deployment-7874f6c9f7-gh6xm   2/2     Running   0          11m
myapp-deployment-7874f6c9f7-nhflg   2/2     Running   0          5s
```