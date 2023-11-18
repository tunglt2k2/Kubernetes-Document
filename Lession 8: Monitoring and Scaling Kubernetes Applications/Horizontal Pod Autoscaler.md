# Exercise: Use the Horizontal Pod Autoscaler (HPA) to automatically scale a Deployment based on CPU utilization.


## 1. Khái niệm

Horizontal Pod Autoscaler điều khiển việc tự động scale theo chiều ngang các Pod, cụ thể là tăng số replica pod khi cao tải và giảm số replica khi tải giảm

HPA đem lại các lợi ích: kinh tế, tự động hóa việc tăng giảm cấu hình hệ thống phù hợp với các hệ thống có khối lượng tải biến đổi nhiều và khó dự đoán.

So với mô hình "truyền thống" kiểu fix cứng số lượng các pods, Auto scaling thích ứng để phù hợp với nhu cầu sử dụng. Chẳng hạn, khi lượng truy cập vào hệ thống vào buổi đêm giảm xuống, các pods có thể được set vào sleep mode 

## 2. Thực hành

**Ta có deploument sau:**
```
apiVersion: apps/v1
  kind: Deployment
  metadata:
   name: php-apache
   namespace: hpa-test
  spec:
   selector:
     matchLabels:
       run: php-apache
   replicas: 1
   template:
     metadata:
       labels:
         run: php-apache
     spec:
       containers:
       - name: php-apache
         image: k8s.gcr.io/hpa-example
         ports:
         - containerPort: 80
         resources:
           limits:
             cpu: 500m
           requests:
             cpu: 200m
  ---
  apiVersion: v1
  kind: Service
  metadata:
   name: php-apache
   namespace: hpa-test
   labels:
     run: php-apache
  spec:
   ports:
   - port: 80
   selector:
     run: php-apache
```

**Tạo HPA bằng lệnh:**
```
kubectl -n hpa-test autoscale deployment php-apache --cpu-percent=50 --min=1 --max=5
```
**hoặc bằng file yaml:**
```
apiVersion: autoscaling/v1
  kind: HorizontalPodAutoscaler
  metadata:
   name: php-apache
   namespace: hpa-test
  spec:
   scaleTargetRef:
     apiVersion: apps/v1
     kind: Deployment
     name: php-apache
   minReplicas: 1
   maxReplicas: 10
   targetCPUUtilizationPercentage: 50
```

**Xem trạng thái ban đầu:**
```
kubectl -n hpa-test get hpa
   
  NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
  php-apache   Deployment/php-apache   0%/50%    1         5         1          17s
```

**Tạo 1 test để kiểm tra scale**
```
kubectl -n hpa-test run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

**Kết quả:**
```
kubectl -n hpa-test get hpa
   
  NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
  php-apache   Deployment/php-apache   211%/50%   1         5         4          10m
```
