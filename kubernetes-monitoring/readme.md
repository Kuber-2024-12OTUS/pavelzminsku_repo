


minikube image build -t nginx:homework .

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm upgrade -i prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --set alertmanager.enabled=false --set grafana.enabled=false -f values.yaml --create-namespace

kubectl port-forward -n monitoring svc/nginx-exp-service 8090:8090
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090




PS C:\Users\admin\OneDrive\Git\Otus_k8s\pavelzminsku_repo\kubernetes-monitoring> kubectl get svc -n monitoring                                                         
NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
nginx-exp-service                       ClusterIP   10.102.229.203   <none>        80/TCP,8090/TCP     8h
prometheus-kube-prometheus-operator     ClusterIP   10.108.249.254   <none>        443/TCP             8h
prometheus-kube-prometheus-prometheus   ClusterIP   10.108.58.202    <none>        9090/TCP,8080/TCP   8h
prometheus-kube-state-metrics           ClusterIP   10.98.221.210    <none>        8080/TCP            8h
prometheus-operated                     ClusterIP   None             <none>        9090/TCP            8h
prometheus-prometheus-node-exporter     ClusterIP   10.100.152.133   <none>        9100/TCP            8h