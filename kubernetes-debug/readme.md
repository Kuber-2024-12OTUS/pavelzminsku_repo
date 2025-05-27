# Выполнено ДЗ № 13

 - [ ] Основное ДЗ
 - [ x] Задание со *

## В процессе сделано:
Задание выполнялось в minikube
1. Создан деплоймент создающий под с контейнером из образа kyos0109/nginx-distroless в неймспейсе default
2. Создан эфемерный контейнер для отладки пода, получены навыки работы с ним согласно заданиям в домашнем задании.
3. Создан отладочный под для ноды на которой запущен под с distroless nginx

## Как запустить проект:
Применяем все манифесты

`kubectl apply -f .`

Проверяем что получилось:

`kubectl get pods -n default -o wide`
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx-distroless-7f6f796cc7-kmxmr   1/1     Running   0          17s   10.244.0.3   minikube   <none>           <none>

Пробрасываем 80й порт на локалхост, проверяем доступность нашего "сайта"

![depl-check](https://github.com/user-attachments/assets/4d19d3c3-a3d9-4738-8e3b-288a07670e38)

Запускаем эфемерный контейнер в поде с nginx, использован отладочный контейнер nicolaka/netshoot

`kubectl debug -it nginx-distroless-7f6f796cc7-kmxmr --image=nicolaka/netshoot --target=nginx --share-processes`

Проверяем доступные процессы

`ps aux`
![ps](https://github.com/user-attachments/assets/7f03aaee-ad5c-4460-b1ad-8e1526a3f0b1)

Выполняем ls -la для для директории /etc/nginx процесса nginx

`/proc/1/root/etc/nginx`

![ls](https://github.com/user-attachments/assets/8e38e5ac-0e5f-4f06-9e10-709904137ac1)

Запускаем tcpdump согласно заданию

`tcpdump -nn -i any -e port 80 `

Обновляем страницу нашего сайта, видим трафик:

![dump](https://github.com/user-attachments/assets/da94e07f-ee38-407d-b5a2-9d064a7a0fc9)

С помощью kubectl debug создадим отладочный под для ноды, на которой запущен под с distroless nginx
Запускать надо с  chroot /host иначе нету доступа к чтению файлов

`kubectl debug node/minikube -it --image=nicolaka/netshoot -- chroot /host`

Читаем логи
`cat /var/log/pods/default_nginx-distroless-*/nginx/0.log`

![log1](https://github.com/user-attachments/assets/9d4ca477-8122-47f3-8646-4a998a8f91b5)

Задание со *
Для выполнения команды strace для процесса nginx в основном контейнере, процесс debug контейнера должен запускаться под профилем sysadmin 
`kubectl debug nginx-distroless-7f6f796cc7-kmxmr -it --image=nicolaka/netshoot --target=nginx --profile=sysadmin `

После запуска смотрим процессы, выполняем strace для воркера, обновляем страничку сайта.

```
ps
strace -p 7

```
![strace](https://github.com/user-attachments/assets/80c05236-2612-4f86-be2e-a4d1ea2818f8)






## PR checklist:
 - [ x] Выставлен label с темой домашнего задания
