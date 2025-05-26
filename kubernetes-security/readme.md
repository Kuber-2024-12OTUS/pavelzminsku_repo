# Выполнено ДЗ №

 - [ x] Основное ДЗ
 - [x ] Задание со *

## В процессе сделано:
В namespace homework создан service account monitoring и ему выдан доступ к эндпоинту /metrics
Работа делалась на minikube и эндпоинта metrics у меня не оказалось, что бы решить данный момент через helm был установлени сервис kube-state-metrics

Изменен манифест deployment, добавлен запуск из-под  service account monitoring

В namespace homework создан service account с именем cd и ему выдана роль admin в рамках namespace homework

Создан  kubeconfig для service account cd и сгенерирован токен для него сроком 1 день.

Модифицирован деплоймент, в инит контейнер добавлен вызов эндпоинта /metrics через wget с сохранение вывода в файл metrics.html который доступен по адресу http://homework.otus/metrics.html

## Как запустить проект:
запускаем minikube, вешаем метку на воркер ноду
`kubectl label nodes minikube env=homework`
Установить сервис kube-state-metrics, если он вдруг отсутствует.
```
helm install kube-state-metrics prometheus-community/kube-state-metrics --namespace kube-system --create-namespace
helm repo update
helm upgrade --install kube-state-metrics prometheus-community/kube-state-metrics --namespace kube-system --create-namespace
```
Должно быть как-то так:
![get-svc](https://github.com/user-attachments/assets/9020d464-c5ab-4d03-b89f-28d5fe463f15)
Создать токен для учетной записи cd
`kubectl create token cd -n homework --duration=24h > token`
добавить содержимое файла в token kubeconfig
добавить в kubeconfig адрес сервера, и корневой сертификат, узнать можно посмотрев конфиг
`kubectl config view`
![image](https://github.com/user-attachments/assets/03052532-504c-4d4a-999e-f6b2798ab8b2)
экспортируем сертификат
`cp d:\.minikube\ca.crt .\ca.crt`
после этого применяем все манифесты из каталога проекта
`kubectl apply -f .\ -R`

## Как проверить работоспособность:
Проверяем что поды запущены:
`kubectl get pods -n homework -o wide`
![image](https://github.com/user-attachments/assets/305bbb5c-869a-48b7-9f72-b918df5f3bb5)
Проверяем сервис аккаунт:
`kubectl describe pod POD_NAME -n homework`
![image](https://github.com/user-attachments/assets/2a8c68bc-5fd5-4497-ba8d-316dece38a26)
Проверяем наличие файла metrics.html по адресу нашего проекта http://homework.otus/metrics.html
![metrics](https://github.com/user-attachments/assets/614df41c-0a7f-48ad-9613-3ad6e60daaef)
Проверяем наличие сервисаккаунтов:
`kubectl get serviceaccounts -n homework`
![image](https://github.com/user-attachments/assets/4ba8d2da-1edc-46eb-98f6-5557c1e7f254)
Проверяем работоспособность kubeconfig, для этого пробуем использовать его в качестве конфига для kubectl
```
kubectl --kubeconfig=./kubeconfig-cd.yaml get pods -n homework
kubectl --kubeconfig=./kubeconfig-cd.yaml get pods -n kube-sytem
```
![image](https://github.com/user-attachments/assets/86e7c183-9808-405f-89ed-cbfc08b9a55d)
Видим что есть доступ к нэймспэйсу homework, а к kube-sytem нету.

## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
