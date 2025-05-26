# Выполнено ДЗ №

 - [ ] Основное ДЗ
 - [ x] Задание со *

## В процессе сделано:
Задание 1.
Создан helm chart позволяющий деплоить приложение созданное в предыдущих заданиях.
Основные параметры в манифесте заданы как переменные в шаблонах и конфигурируются через values.yaml
Репозиторий и тег образа разные параметры
Пробывключаемы/отключаемы через конфиг
В notes описано сообщение после установки релиза, отображающее адрес, по которому можно обратится к сервису
В чарт добавлена зависомость - mysql
Задание 2.
Создан helmfile позволяющий запускать 2 экзэмпляра kafka со следующими параметрами:
Prod: 
Установлен в namespace prod 
Должно быть развернуто 5 брокеров 
Должна быть установлена kafka версии 3.5.2 
Для клиентских и межброкерных взаимодействий должен использоваться протокол SASL_PLAINTEXT
Dev:
Установлен в namespace dev 
Должно быть развернут 1 брокер 
Должна быть установлена последняя доступная версия kafka 
Для клиентских и межброкерных взаимодействий должен использоваться протокол PLAINTEXT, авторизация для подключения к кластеру отключена

## Как запустить проект:
Установить сервис kube-state-metrics, если он вдруг отсутствует.
```
helm install kube-state-metrics prometheus-community/kube-state-metrics --namespace kube-system
helm repo update
helm upgrade --install kube-state-metrics prometheus-community/kube-state-metrics --namespace kube-system
```
из директории проекта выполнить:
`helm install relese1 . -n homework --create-namespace`

для выполнения задания 2
установить helmfile
под win: 
`scoop install helmfile`
подключить vpn если репозиторий bitnami недоступен
выполнить
` helmfile -f helmfile.yaml sync`



## Как проверить работоспособность:
Пройти по адресу который будет выведен после успешной установки из чарта, не забыв сделать tunnel или порт форвардинг через k9s

http://homework.otus
![check1](https://github.com/user-attachments/assets/399cf09a-8374-4d4d-8b71-ef7b8c3ff1e1)

можно проверить что доступны метрики
![check_metrics](https://github.com/user-attachments/assets/14bfd855-96a0-46fd-8d58-941c23e2cbd3)

Проверить что установлени и запущен pod mysql
![checkmysql](https://github.com/user-attachments/assets/7c0b7001-179c-464f-91f4-932031f30fc4)

После успесной установки kafka будет вывод:
![kafkarelease](https://github.com/user-attachments/assets/3b95b2d8-c897-4ebb-ae02-c882ea6015e6)
Можно удостовериться что dev и prod запущены с нужными параметрами
Dev
![dev](https://github.com/user-attachments/assets/e331e6c9-fee6-47c1-86ea-8af7d0154017)
Prod
![prod](https://github.com/user-attachments/assets/a7cba7f0-041c-4ccf-9e1d-5abadf476108)



## PR checklist:
 - [x] Выставлен label с темой домашнего задания
