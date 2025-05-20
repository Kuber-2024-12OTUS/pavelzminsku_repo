# Выполнено ДЗ №

 - [ ] Основное ДЗ
 - [ х] Задание со *

## В процессе сделано:
Создан манифест объекта CustomResourceDefinition со следующими параметрами: 
● Объект уровня namespace 
● Api group – otus.homework 
● Kind – MySQL 
● Plural name – mysqls 
● Версия – v1 
У объекта должны быть следующие обязательные атрибуты и правила их валидации(все поля строковые): 
● Image – определяет docker-образ для создания 
● Database – имя базы данных 
● Password – пароль от БД 
●Storage_size – размер хранилища под базу
 Создан манифест  описывающий сервис аккаунт с полными правами на доступ к api серверу 
 Создан манифест deployment для оператора
 Создан манифест кастомного объекта kind: MySQL валидный для применения (см. атрибуты CRD созданного ранее) 
Для задания со *
Изменены права для сервисаккаунта, оставлены минимально необходимые права.

## Как запустить проект:
Применить все манифесты, из каталога выполнить команду
`kubectl apply -f .\ -R`

## Как проверить работоспособность:
Проверяем что все ресурсы созданы:
`kubectl get crd,deployment,pods,svc,pvc,pv`

![crd-check](https://github.com/user-attachments/assets/98d8b2e5-769a-4ea9-94b1-dd0b5082aeef)

Удаляем mysql, проверяем что все связанные с ним ресурсы
`kubectl delete mysql mysql1`
`kubectl get pods,svc,pvc,pv`

![deelete-pods-check](https://github.com/user-attachments/assets/7b3dc60c-9e3e-4f56-a736-e7143cb39aa6)

Проверяем набор прав у сервисаккаунта sa-mysql
`kubectl auth can-i --list --as=system:serviceaccount:default:sa-mysql`

![rbac check](https://github.com/user-attachments/assets/5cd5dcd9-9a26-4055-8da1-914cc7d0dd4a)

проверяем что будет если попробовать превысить права, например удалить из-под него pod

![delete-check](https://github.com/user-attachments/assets/5f28f137-2a06-47ef-850a-a4f2cb63efcd)

Прав не хватает.


## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
