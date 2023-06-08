Trên bài lab này mình lấy metric của node nên mình sẽ cài đặt node Exporter.
Tải :
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
```
Giải nén:
```
tar xvf node_exporter-1.3.1.linux-amd64.tar.gz
```
Tiến hành coppy vào bin:
```
cd node_exporter-1.3.1.linux-amd64
sudo cp node_exporter /usr/local/bin
```
Tiến hành tạo user node-exporter.
```
sudo useradd --no-create-home --shell /bin/false node_exporter
```
Phân quyền:
```
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
Tạo service cho node exporter.
```
sudo vi /etc/systemd/system/node_exporter.service
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
Khởi động :
```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
```
Nếu bật tường lửa t cần mở port:
```
sudo ufw allow 9100
```
Đã xong giờ kết nối đến Prometheus nữa là có thể lấy được metric của node.
