Triển khai grafana không có gì đặc biệt, chúng ta sẽ để mặc định tất cả. Pasword khi login lần đầu sẽ đổi pass luôn.
dockerfile:
```
FROM grafana/grafana:8.0.6
EXPOSE 3000
CMD ["grafana-server", "--config=/etc/grafana/grafana.ini"]
```
