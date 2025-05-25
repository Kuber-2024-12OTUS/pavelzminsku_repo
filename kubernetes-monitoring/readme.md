# Выполнено ДЗ №

 - [ ] Основное ДЗ
 - [ x] Задание со *

## В процессе сделано:
Собран кастомный образ nginx отдающий свои метрики на эндпоинте /nginx_metrics
Установлен Prometheus
Настроен prometheus export для сбора метрик с nginx
Создан servicemonitor описывающий сбор метри созданых подов

## Как запустить проект:
Во всех основных образах nginx module http_stub_status_module уже включен в образ.

minikube image build -t nginx:homework .


## Как проверить работоспособность:
 - Например, перейти по ссылке http://localhost:8080

## PR checklist:
 - [ ] Выставлен label с темой домашнего задания



minikube image build -t nginx:homework .

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm upgrade -i prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --set alertmanager.enabled=false --set grafana.enabled=false -f values.yaml --create-namespace

kubectl port-forward -n monitoring svc/nginx-exp-service 8090:8090
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090
