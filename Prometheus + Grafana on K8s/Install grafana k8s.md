Để triển khai grafana trên k8s làm theo bước sau:
B1: Tạo namespace, ở đây mình tạo namespace với tên ``` monitoring```
```
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```
Bước 2: tạo file deployment cho grafana
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:8.0.6
          ports:
            - containerPort: 3000
          env:
            - name: GF_SECURITY_ADMIN_USER
              value: admin
            - name: GF_SECURITY_ADMIN_PASSWORD
              value: admin
      volumes:
        - name: grafana-storage
          emptyDir: {}
```
- Ở đây mình dùng image của grafana luôn với version 8.0.6
- Khai báo user và pass. Hiện mình để admin khi đăng nhập vào Ui grafana sẽ yêu cầu bạn đổi pass luôn.
Bước 3: Tạo service
```
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    app: grafana
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30012
  type: NodePort
  ```
  Ở đây mình dùng NodePort để có thể truy cập từ ngoài qua port 30012.
