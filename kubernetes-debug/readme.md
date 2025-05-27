kubectl apply -f .



kubectl get pods -n default -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx-distroless-7f6f796cc7-kmxmr   1/1     Running   0          17s   10.244.0.3   minikube   <none>           <none>


kubectl debug -it nginx-distroless-7f6f796cc7-kmxmr --image=nicolaka/netshoot --target=nginx --share-processes

ps aux

ls -la /proc/1/root/etc/nginx

tcpdump -nn -i any port 80

kubectl debug node/minikube -it --image=nicolaka/netshoot -- chroot /host

cat /var/log/pods/default_nginx-distroless-*/nginx/0.log

kubectl debug nginx-distroless-7f6f796cc7-kmxmr -it --image=nicolaka/netshoot --target=nginx --profile=sysadmin 

ps
strace -p 7