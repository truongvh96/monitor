Update và cài đặt các gói bổ trợ.
```
sudo apt update -y

sudo apt install python3 curl net-tools git wget python3-pip -y
```
 Cài đặt & cấu hình vmware_exporter
 ```
pip3 install --ignore-installed PyYAML

pip3 install vmware_exporter
```
Di chuyển vào thư mục chưa file cài đặt vmware_exporter.
```
cd /usr/local/lib/python*/dist-packages/vmware_exporter
```
Tạo file tên là config.yml trong thư mục /usr/local/lib/python*/dist-packages/vmware_exporter bằng lệnh
```
touch config.yml
```
Passte nội dung cho file config.yml như bên dưới (có thể sử dụng lệnh echo ở dưới để tạo file này.)
```
default:
    vsphere_host: x.x.x.x
    vsphere_user: 'administrator@vsphere.local'
    vsphere_password: 'mat_khau_vcenter'
    ignore_ssl: True
    specs_size: 5000
    fetch_custom_attributes: True
    fetch_tags: True
    fetch_alarms: True
    collect_only:
        vms: True
        vmguests: True
        datastores: True
        hosts: True
        snapshots: True"
 ```
Trong dó lưu ý thay đổi các tham số:

- vsphere_host: IP của vCenter. Bạn hoàn toàn có thể khai báo IP của host ESXi mà bạn có để thay cho vCenter nhé.
- vsphere_user: Tên đăng nhập
- vsphere_password: Mật khẩu

Tạo user để phân quyền cho vmware_exporter
```
useradd -M -r -s /bin/false vmware_exporter
```
Thực hiện phân quyền file cấu hình và file thực thi.
```
chown -R vmware_exporter:vmware_exporter /usr/local/lib/python*/dist-packages/vmware_exporter

chown vmware_exporter:vmware_exporter /usr/local/bin/vmware_exporter
```
Sử dụng lệnh cat dưới đây để tạo file service cho vmware_exporter.
```
cat <<EOF > /etc/systemd/system/vmware_exporter.service
[Unit]
Description=VMWare Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=vmware_exporter
Group=vmware_exporter
Type=simple
ExecStart=/usr/bin/python3 /usr/local/bin/vmware_exporter -c /usr/local/lib/python3.8/dist-packages/vmware_exporter/config.yml
ExecReload=/bin/kill -HUP \$MAINPID
SyslogIdentifier=vmware_exporter
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
Khởi động vmware_exporter và kiểm tra trạng thái.
```
systemctl daemon-reload

systemctl start vmware_exporter

systemctl enable vmware_exporter

systemctl status vmware_exporter
```
Kiểm tra xem port 9272 mà vmware_exporter được chạy hay chưa.
```
ss -lah | egrep 9272
```
Bạn có thể dùng vi/vim sửa file /etc/prometheus/prometheus.yml để thêm đoạn cấu hình dưới hoặc dùng lệnh echo dưới để bổ sung vào dưới cùng của file cấu hình prometheus.
```
echo "
  ## Khai bao target cho vcenter
  - job_name: 'vcenter_node'
    scrape_timeout: 15s
    scrape_interval: 15s
    metrics_path: '/metrics'
    static_configs:
      - targets:
        - '<ip vcenter>' 
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <ip_prometheus:9272>" >> /etc/prometheus/prometheus.yml
 ```
 Khởi động lại prometheus
 ```
 systemctl restart prometheus.service
 ```
 Truy cập vào địa chỉ: http://<ip_prometheus>:9090/ sau đó vào tab Status ==> Targets hoặc vào thẳng địa chỉ http://<ip_prometheus>:9090/targets
