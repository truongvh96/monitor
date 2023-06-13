Bước 1: Download wmi_exporter (bản mới nhất thì nó sẽ được đổi tên thành windows_exporter). Anh em chú ý để edit trong file dashboard nha. 


Link download Agent: https://github.com/martinlindhe/wmi_exporter/releases

Có 2 phiên bản Agent:

wmi_exporter.exe (click to run, dành cho các bạn nào chỉ cần các metric được enable sẵn).

wmi_exporter.msi (dùng để cài đặt thông qua CMD, enable các tính năng thu thập metric nâng cao).

Mở port 9182 trên windows server.


Bước 2: Mở CMD với quyền Administrator.


Bước 3: Chạy comment msiexec với cú pháp như sau:

```
msiexec /i C:\wmi_exporter-0.9.0-amd64.msi ENABLED_COLLECTORS="ad,cpu,cs,logon,memory,logical_disk,os,service,system,process,tcp,net,textfile,thermalzone"
```
Trong đó C:\wmi_exporter-0.9.0-amd64.msi là đường dẫn chứa file wmi_exporter.msi.

ENABLED_COLLECTORS=”các loại metric cần thu thập, tên của loại metric trong hình bên trên”

Bước 4: Kiểm tra metric bạn truy cập như sau: http://ipserver:9182

Bước 5: Tạo job trong prometheus để giám sát Windows Server này với nội dung sau:
 ```
 Vi /usr/local/prometheus/prometheus.yml
 ```
 Thêm nội dung sau:
 ```
  - job_name: 'windows'
    scrape_interval: 10s
    static_configs:
    - targets: 
      - '10.10.10.8:9182'
      labels:                           
       hostname: DC01
       type: windows
       company: XYZDG
```
Bước 6: Restart serivce prometheus
```
systemctl restart prometheus
systemctl status prometheus
```


**Sau khi cài đặt VMI Exporter trên win server bạn cần khai báo vào configmap prometheus server.**
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
      - job_name: 'vmi windowns server'
        static_configs:
          - targets: ['url:9090']
```
