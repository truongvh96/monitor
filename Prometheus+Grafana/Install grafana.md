Hướng dẫn cài đặt Grafana trên Ubuntu 20.04

Thêm khóa GPG Grafana trong Ubuntu bằng Curl:
```
curl  -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add –
```
thêm source
```
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
Tiến hành update:
```
sudo apt-get update
```
Sau khi update tiến hành cài đặt Grafana:
```
sudo apt-get install grafana
```
Start grafana:
```
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server.service
```
Đã xong giờ ta truy cập vaafo UI của grafana theo đường link sau: ``` http://your_ip:3000 ```

Tài khoản và mật khẩu mặc định là  ``` admin/admin``` khi truy cập vào grafana sẽ bắt chúng ta đổi mật khẩu ngay. Lúc này bạn đổi mật khẩy để bảo mật.

![image](https://github.com/truongvh96/monitor/assets/97424062/fa8cd543-6d65-4ed5-bd1e-8eefc154e88d)

![image](https://github.com/truongvh96/monitor/assets/97424062/47e07083-a1bc-47e6-9096-3f8d2a799b4c)

Ui grafana sẽ như thế này:

![image](https://github.com/truongvh96/monitor/assets/97424062/3fc6c4b6-ddb1-4df0-88da-0cb225267b71)

