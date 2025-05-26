# Выполнено ДЗ №

 - [ x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
Домашнее задание выполнено в Managed Service for Kubernetes в Yandex Cloud
Установлен Argocd
С помощью atgocd развернуто 2 приложения bernetes-networks и kubernetes-templating согласно требования ДЗ

## Как запустить проект:
Проверяем ноды на предмет наличия корректных taint и labels, на инфрастукрутной ноде должен быть taint node-role=infra:NO_SCHEDULE   на воркер ноде env=homework, для корректного запуска приложения.
```
kubectl get nodes
kubectl get nodes --show-labels | grep infra
kubectl get nodes --show-labels | grep homework
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
kubectl label node <node name> env=homework
```
Устанавливаем argocd
```
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd -n argocd --create-namespace -f .\argo-values.yaml
```
Получаем пароль от учетной записи admin
`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

делаем port-forward для доступа в argocd

`kubectl port-forward service/argocd-server -n argocd 80:443`

для корректной работы приложения kubernetes-templating надо установить kube-state-metrics
```
helm install kube-state-metrics prometheus-community/kube-state-metrics --namespace kube-system
helm repo update
helm upgrade --install kube-state-metrics prometheus-community/kube-state-metrics --namespace kube-system
```
так же понадобилось обновить storage class, домашнее задание по темплейтингу делалось в миникуб, коммитим в мастер изменения, меняем содержимое sc-homework.yaml
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sc-homework
provisioner: disk-csi-driver.mks.ycloud.io  
parameters:
  type: network-hdd  
reclaimPolicy: Retain
```
Применяем манифесты:
kubectl apply -f .\argo-project.yaml
kubectl apply -f .\argo-app-helm.yaml
kubectl apply -f .\argo-app-man.yaml

## Как проверить работоспособность:
Результат проверки корректности taint и label

![nodes-check](https://github.com/user-attachments/assets/4a9120a8-07a2-4e7c-a0b4-08f9b4ffa70b)

Проверяем что argocd корректно установлен на нужную ноду

![argo-check2](https://github.com/user-attachments/assets/bcb4957f-93b9-427e-811e-799946690803)

Проверяем что приложения установлены в нужные namespace и на нужный воркер
`kubectl get all -n homework -o wide`

![homework](https://github.com/user-attachments/assets/34ca7366-43b7-42a4-bb4c-fffc435bd48d)

` kubectl get all -n homeworkhelm - o wide`

![homeworkhelm](https://github.com/user-attachments/assets/5f8409fb-e0b2-4373-a300-8c1a7142bfe0)

Заходим в argocd проверяем там приложения:

![argo-temp](https://github.com/user-attachments/assets/122aaf3c-55fa-43c2-b304-033c2ab8d466)

![argo-net](https://github.com/user-attachments/assets/d1e205e2-4e09-4b92-8419-8fb411d898ce)



## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
