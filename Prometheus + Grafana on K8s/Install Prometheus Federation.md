Federation là tính năng cho phép một Prometheus-server scrape metrics từ các Prometheus-server khác về và lưu trữ metrics.

Để triển khai Federation trên k8s t làm như sau:

Bước 1:  Tạo configmap cho Federation
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-federation-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'federate'
        metrics_path: '/federate'
        params:
          'match[]':
            - '{job=~"prometheus|node-exporter|vmware|kube-state-metrics"}'
            - '{__name__=~"job:.*"}'
        static_configs:
          - targets:
              - 'prometheus-service.monitoring:9090'
              - '172.24.109.195:9090'
```
- Lưu ý: Job phải đúng với job bạn đã khai báo ỏ prometheus server. Target bạn chỉ định đến prometheus server.

Chạy lệnh sau để tạo configmap
```
kubectl apply -f configmap-federation.yaml -n monitoring
```

Bước 2: Tạo deployment cho federation
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-federation
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-federation
  template:
    metadata:
      labels:
        app: prometheus-federation
    spec:
      containers:
        - name: prometheus-federation
          image: prom/prometheus:v2.30.3
          args:
            - --config.file=/etc/prometheus/prometheus.yml
            - --web.enable-admin-api
          ports:
            - containerPort: 9090
          volumeMounts:
            - name: prometheus-federation-config
              mountPath: /etc/prometheus
      volumes:
        - name: prometheus-federation-config
          configMap:
            name: prometheus-federation-config
 ```
 Bước 3: Tạo service cho federation
 ```
 apiVersion: v1
kind: Service
metadata:
  name: prometheus-federation
  namespace: monitoring
spec:
  selector:
    app: prometheus-federation
  ports:
    - protocol: TCP
      port: 9090
      targetPort: 9090
      nodePort: 30013
  type: NodePort
  ```
  - Tại đây mình dùng Nodeport để truy cập vào UI cho tiện.
  - Sau khi đã đủ thành phần t cahjy lệnh sau để apply
  ```
  kubectl apply -f prometheus-federation.yaml -n monitoring
  ```
