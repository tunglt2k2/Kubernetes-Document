# **Execises: Set up Prometheus and Grafana to monitor your Kubernetes cluster.**

## **1. Set up Prometheus**

### **Bước 1: Cài Helm**

Helm là một trình quản lý gói và công cụ quản lý ứng dụng cho Kubernetes, nó đóng gói nhiều tài nguyên Kubernetes vào một đơn vị triển khai logic duy nhất được gọi là Chart. Bên trong của Chart sẽ có phần chính là các "template, là định nghĩa các tài nguyên sẽ triển khai lên k8s.

**Cài helm cho kubectl:**
```
sudo snap install helm --classic
```

### **Bước 2: deploy prometheus**

**Dowload repo deploy cho prometheus bằng helm:**
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
**Install repo:**
```
helm install prometheus prometheus-community/prometheus
```
**Kết quả tương tự như sau:**

```
NAME                                                    READY   STATUS    RESTARTS   AGE
pod/prometheus-alertmanager-0                           1/1     Running   0          59s
pod/prometheus-kube-state-metrics-86894fb745-6skdg      1/1     Running   0          59s
pod/prometheus-prometheus-node-exporter-jhk9h           1/1     Running   0          59s
pod/prometheus-prometheus-pushgateway-7cb5dd797-tmzjt   1/1     Running   0          59s
pod/prometheus-server-76cf698f7f-8pptg                  1/2     Running   0          59s

NAME                                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/kubernetes                            ClusterIP   10.96.0.1        <none>        443/TCP    144m
service/prometheus-alertmanager               ClusterIP   10.104.101.202   <none>        9093/TCP   59s
service/prometheus-alertmanager-headless      ClusterIP   None             <none>        9093/TCP   59s
service/prometheus-kube-state-metrics         ClusterIP   10.96.17.111     <none>        8080/TCP   59s
service/prometheus-prometheus-node-exporter   ClusterIP   10.105.82.31     <none>        9100/TCP   59s
service/prometheus-prometheus-pushgateway     ClusterIP   10.100.166.112   <none>        9091/TCP   59s
service/prometheus-server                     ClusterIP   10.108.226.119   <none>        80/TCP     59s

NAME                                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/prometheus-prometheus-node-exporter   1         1         1       1            1           kubernetes.io/os=linux   59s

NAME                                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-kube-state-metrics       1/1     1            1           59s
deployment.apps/prometheus-prometheus-pushgateway   1/1     1            1           59s
deployment.apps/prometheus-server                   0/1     1            0           59s

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-kube-state-metrics-86894fb745      1         1         1       59s
replicaset.apps/prometheus-prometheus-pushgateway-7cb5dd797   1         1         1       59s
replicaset.apps/prometheus-server-76cf698f7f                  1         1         0       59s

NAME                                       READY   AGE
statefulset.apps/prometheus-alertmanager   1/1     59s
```

### **Bước 3: Expose prometheus ra ngoài**
Tạo một service mới có type là Nodeport giống service prometheus-server 
```
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
```
Mở service trên trình duyệt: 
```
minikube service prometheus-server-ext
```

## **2. Set up Grafana**


### **Bước 1: Deploy prometheus**

**Dowload repo deploy cho grafana bằng helm và cài đặt:**

- Dowload
    ```
    helm repo add grafana https://grafana.github.io/helm-charts
    ```
-Install
    ```
    helm install grafana grafana/grafana
    ```
### **Bước 2: Expose grafana ra ngoài**
Tạo một service mới có type là Nodeport giống service grafana
```
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
```
Mở service trên trình duyệt: 
```
minikube service grafana-ext
```
### **Bước 3: Xem tài khoàn, mật khẩu của grafana**
```
tung@tung-ideapad-5-pro:~$ kubectl get secret --namespace default grafana -o yaml
apiVersion: v1
data:
  admin-password: V25hdzIyeGhlc21JQnBPNWN3bmk4R1FWNGE3SUtqVUJQb21oMjhSTg==
  admin-user: YWRtaW4=
  ldap-toml: ""
kind: Secret
metadata:
  annotations:
    meta.helm.sh/release-name: grafana
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2023-07-09T18:43:06Z"
  labels:
    app.kubernetes.io/instance: grafana
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: grafana
    app.kubernetes.io/version: 9.5.5
    helm.sh/chart: grafana-6.57.4
  name: grafana
  namespace: default
  resourceVersion: "8623"
  uid: 54100753-2db2-4a81-b056-db1dd980b5ac
type: Opaque 
tung@tung-ideapad-5-pro:~$ echo "V25hdzIyeGhlc21JQnBPNWN3bmk4R1FWNGE3SUtqVUJQb21oMjhSTg==
" | openssl base64 -d ; echo
Wnaw22xhesmIBpO5cwni8GQV4a7IKjUBPomh28RN
tung@tung-ideapad-5-pro:~$ echo "YWRtaW4=" | openssl base64 -d ; echo                    
admin
```

