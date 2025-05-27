# Выполнено ДЗ №

 - [ x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
Настроен managed k8s в Yandex cloud
Создан serviceaccount с правами на создание и изменение бакетов s3
Сгенерированы ключи доступа
Создан манифест PVC, использующий для хранения созданный вами storageClass с механизмом autoProvisioning (тут немного не понял, автоприженинг предполагает автоматические создание бакетов, тогда как в ДЗ прописано ручное создание бакета, в дальнейшем оталкивался от требования автопроженинга)
 
## Как запустить проект:
 Устанавливаем CSI драйвер

```
git clone https://github.com/yandex-cloud/k8s-csi-s3.git
kubectl apply -f k8s-csi-s3/deploy/kubernetes/

```
![2025-05-27 02_37_35-Window](https://github.com/user-attachments/assets/e8565c71-7331-465d-b15d-8e092421da67)

Применяем все манифесты
`kubectl apply -f .`

## Как проверить работоспособность:
Проверяем pvc

`kubectl get pvc s3-pvc
`
![pvc-checl](https://github.com/user-attachments/assets/444c936a-80e7-42c4-845a-7ce56a4fad44)

Проверяем pod

`kubectl get pods -n default`

![pod-check](https://github.com/user-attachments/assets/861e184a-b77a-4e42-8604-cdca85bd590e)

Проверяем наличие файлов в бакете s3

![s3-check](https://github.com/user-attachments/assets/1292e54d-4035-43e4-a6b4-fc3c269be4ad)


## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
