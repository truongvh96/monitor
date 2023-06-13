Triển khai Prometheus server để thu thập metric từ các exporter khác.

Bước 1:
- Trước tiên bạn cần tạo configmap cho prometheus server.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      - job_name: 'vmware'
        static_configs:
          - targets: ['prometheus-service.monitoring:9090']
      - job_name: 'kube-state-metrics'
        static_configs:
          - targets: ['kube-state-metrics-service:8080']
      - job_name: 'node-exporter'
        static_configs:
          - targets: ['node-exporter-service:9100']
```
- Ta cần khai báo job với target của các exporter muốn thu thập metric.

Bước 2: tạo file deployment cho prometheus server 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-server
  template:
    metadata:
      labels:
        app: prometheus-server
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus:v2.30.3
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus"
          ports:
            - name: web
              containerPort: 9090
          volumeMounts:
            - name: prometheus-config
              mountPath: /etc/prometheus
            - name: prometheus-storage
              mountPath: /prometheus
      volumes:
        - name: prometheus-config
          configMap:
            name: prometheus-config
        - name: prometheus-storage
          emptyDir: {}
```
- Ở đây ta khai báo args sử dụng config tại file ``` /etc/prometheus/prometheus.yml ```

Bước 3: Tạo serrvice cho prometheus server 
```
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
spec:
  selector:
    app: prometheus-server
  ports:
    - name: web
      port: 9090
      targetPort: 9090
```
- Prometheus server sẽ sử dụng port 9090

- Chạy lệnh sau để apply
```
kubectl apply -f prometheus-server.yaml -n monitoring
```
