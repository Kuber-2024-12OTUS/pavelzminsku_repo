# Выполнено ДЗ №

 - [ ] Основное ДЗ
 - [x ] Задание со *

## В процессе сделано:
Создан кластер k8s с использование kubeadm
Обновлен кластер k8s на новую версию с использвоание kubeadm
Установлен кластре k8s с использование kubespray

## Как запустить проект:

Создать 4 сервера, (дз выполнялось в YC ) 1 мастер ноду и 3 воркер ноды, настроена авторизация по ключу в свойствах вм.

![yc](https://github.com/user-attachments/assets/1e5ebe93-8ec8-4b1e-9f01-42bd7bf2ac7e)

Готовим все сервера для установки:

```
#отключаем фаервол
systemctl stop ufw && systemctl disable ufw
#отключаем swap
sudo swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

```
# Загрузка модулей ядра
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

```
#настраиваем сеть
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```
Устанавливаем containerd
```
sudo apt-get update
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
#Включаем использование systemdcgroup
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
```

Устанавливаем k8s:
Добавляем репозиторий предпоследней версии k8s (на момент выполнения актуальная 1.33)
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Ставим kubelet kubeadm kubectl
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
![install](https://github.com/user-attachments/assets/22e9b17b-d573-42db-80b7-a7fb2ac9c36a)

Переходим к мастер ноде, выполняем инициализацию на мастер ноде

`sudo kubeadm init --pod-network-cidr 10.244.0.0/16`

копируем конфиги
```
mkdir -p $HOME/.kube && \ 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && \
sudo chown $(id -u)\:$(id -g) $HOME/.kube/config
```

Не забываем сохранить токен для добавления нод в кластер, он понадобится в дальнейшем.
```
kubeadm join 10.130.0.3:6443 --token <token> \
	--discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 
```

Устанавливаем flannel

`kubectl apply -f https://github.com/flannel-io/flannel/releases/download/v0.26.2/kube-flannel.yml`

Переходим к worker тодам, добавляем их в кластер с помощи полученого ранее токена:

```
kubeadm join 10.130.0.3:6443 --token <token> \
	--discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 
```
  
На мастер ноде добавляем label для воркеров.
```
kubectl label node worker1 node-role.kubernetes.io/worker=worker && \
kubectl label node worker2 node-role.kubernetes.io/worker=worker && \
kubectl label node worker3 node-role.kubernetes.io/worker=worker

```
Проверяем что получилось:
`kubectl get nodes -A -o wide`
Видим что на всех нодах версия 1.32

![nodes-32](https://github.com/user-attachments/assets/6bba0618-ec87-434b-9ccb-cd025c7ae86a)

![pods](https://github.com/user-attachments/assets/f07fa098-a911-4558-9185-5b2a663e9199)

Переходим к процедуре обновления. 
Сначала надо обновить мастер ноду

Добавляем репо версии 1.33	
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Обновляем kubeadm
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.1-1.1' && \
sudo apt-mark hold kubeadm
```

Обновляем мастер ноду:
```
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.33
```
`kubectl drain master1 --ignore-daemonsets`

```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.1-1.1' kubectl='1.33.1-1.1' && \
sudo apt-mark hold kubelet kubectl
```

```
sudo systemctl daemon-reload && \
sudo systemctl restart kubelet
```

`kubectl get nodes`

![master-33](https://github.com/user-attachments/assets/0bdeb1e7-e7e7-413c-a43b-2ac2402b3d81)

Мастер обновлен.

Переходим к обновления воркеров, следующий блок повторить для каждого

На мастере выполянем
`kubectl drain worker1 --ignore-daemonsets`

Обновляем:

`echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list`

```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.1-1.1' && \
sudo apt-mark hold kubeadm
```

`sudo kubeadm upgrade node`

```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.1-1.1' kubectl='1.33.1-1.1' && \
sudo apt-mark hold kubelet kubectl

```
На мастере:
`kubectl uncordon worker1`

Проверяем результат:
`kubectl get nodes -A -o wide`

![nodes-33](https://github.com/user-attachments/assets/efad69a5-6fe1-4931-a943-32f457ba0135)
 Обновление завершено

==============================================

Задание со *
 
Создаем новые сервера, 3 мастер ноды, 2 воркер ноды

![instance2](https://github.com/user-attachments/assets/80c17f52-e201-4c31-812e-9336d3f83114)

Готовим inventory для kubespray

Устанавливаем ansible, я выполнял в wsl ubuntu.
Lля kubespray надо версия ниже, чем имеющаяся в репо, ставил из pip

```
sudo apt update
sudo apt install python3-pip python3-venv -y

```
```
python3 -m venv ansible-env
source ansible-env/bin/activate
```

```
 pip install ansible-core==2.16.6
 pip install ansible==9.5.1
 pip install netaddr
```
Ставим kubespray

```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
```
`sudo pip install -r requirements.txt`

Запускаем плэйбук с нашим inventory.ini. авторизация по ssh ключу 

`ansible-playbook -i inventory/inventory.ini cluster.yml -b -u administrator`

Ждем минут 10 пока завершится установка, проверяем результат

`sudo kubectl --kubeconfig=/etc/kubernetes/admin.conf get pod -o wide -A`

![kunespray_result](https://github.com/user-attachments/assets/a196d2e2-4317-47e0-aca8-c013a9b858ca)

![kubespray-podes](https://github.com/user-attachments/assets/8c0b4eac-9011-4c6c-89b6-720caf99cea2)





## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
