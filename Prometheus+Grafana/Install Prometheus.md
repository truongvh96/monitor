Trước tiên bạn cần tạo user:
```
sudo useradd --no-create-home --shell /bin/false prometheus  
sudo useradd --no-create-home --shell /bin/false node_exporter
```
Tạo folder cho prometheus:
```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
Phân quyền cho 2 thư mục trên:
```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```
Xong bước chuẩn bị. Cài đặt prometheus thôi.
```
cd /opt/
wget https://github.com/prometheus/prometheus/releases/download/v2.26.0/prometheus-2.26.0.linux-amd64.tar.gz
```
Sau khi tải về, giải nén ra.
```
tar -xvf prometheus-2.26.0.linux-amd64.tar.gz
cd prometheus-2.26.0.linux-amd64
```
Ta coppy vào /usr/local/bin/
```
sudo cp /opt/prometheus-2.26.0.linux-amd64/prometheus /usr/local/bin/
sudo cp /opt/prometheus-2.26.0.linux-amd64/promtool /usr/local/bin/
```
Phân quyền cho 2 thư mục mới:
```
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
```
Coppy console vào Lib
```
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/consoles /etc/prometheus
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/console_libraries /etc/prometheus
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/prometheus.yml /etc/prometheus
```
Phân quyền cho 3 thư mục trên:
```
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /etc/prometheus/prometheus.yml
```
Cấu hình:
```
cat /etc/prometheus/prometheus.yml
```
```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
            - targets: ['localhost:9090','<ip_node_exporter>:9100']

  ## Khai bao target cho vcenter
  - job_name: 'vcenter_node'
    scrape_timeout: 15s
    scrape_interval: 15s
    metrics_path: '/metrics'
    static_configs:
      - targets:
        - '<ip_vmware>'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <ip_prometheus_server>:9272
  - job_name: 'windows'
    static_configs:
            - targets: ['<ip_win>:9182']
```
Chú ý: với target t sẽ khai báo thêm các node exporter sau.

Tạo file service cho prometheus
```
vi /etc/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
Xong rồi khởi động lên và kiểm tra thôi:
```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```
Truy cập qua port 9090 để vào UI của prometheus:
```
http://server-IP-or-Hostname:9090
```
Giao diện sẽ như sau:

![image](https://github.com/truongvh96/monitor/assets/97424062/ce65d009-6157-4c7e-8f18-e35ffdc6b188)

Vào target sẽ hiển thị tất cả các node export đã kết nối đến prometheus:

![image](https://github.com/truongvh96/monitor/assets/97424062/52ee2a92-0cce-4d05-bdd4-0b74d718f09f)

