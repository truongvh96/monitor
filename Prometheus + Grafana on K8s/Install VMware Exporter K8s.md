Để triển khai Vmware Exporter trên k8s t làm như sau:

Bước 1:  Tạo configmap cho VMware Exporter
```
kind: ConfigMap
metadata:
  labels:
    app: vmware-exporter
  name: vmware-exporter-config
apiVersion: v1
data:
  VSPHERE_USER: "uer"
  VSPHERE_HOST: "x.x.x.x"
  VSPHERE_IGNORE_SSL: "True"
  VSPHERE_COLLECT_HOSTS: "True"
  VSPHERE_COLLECT_DATASTORES: "True"
  VSPHERE_COLLECT_VMS: "True"
```
- Lưu ý: ở đây bạn cần khai báo host và user của VMware 

Chạy lệnh sau để tạo configmap
```
kubectl apply -f configmap-vmware.yaml -n monitoring
```

Bước 2: Tạo deployment cho VMware Exporter
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vmware-exporter
spec:
  selector:
    matchLabels:
      app: vmware-exporter
  template:
    metadata:
      labels:
        app: vmware-exporter
        release: vmware-exporter
      annotations:
        prometheus.io/path: "/metrics"
        prometheus.io/port: "9272"
        prometheus.io/scrape: "true"
    spec:
      containers:
      - name: vmware-exporter
        image: "pryorda/vmware_exporter:latest"
        imagePullPolicy: Always
        ports:
        - containerPort: 9272
          name: http
        envFrom:
        - configMapRef:
            name: vmware-exporter-config
        - secretRef:
            name: vmware-exporter-password
 ```
 Bước 3: Tạo service cho VMware Exporter
 ```
apiVersion: v1
kind: Service
metadata:
  name: vmware-exporter-service
  labels:
    app: vmware-exporter
spec:
  selector:
    app: vmware-exporter
  ports:
    - name: http
      protocol: TCP
      port: 9272
      targetPort: 9272
  ```
  - Sau khi đã đủ thành phần ta chạy lệnh sau để apply
  ```
  kubectl apply -f vmware-exporter.yaml -n monitoring
  ```
