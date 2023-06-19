Để build image với config ta cần tạo 1 file config ``` prometheus-federtion.yaml ```
```
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
Sau khi đã có file config tạo dockerfile thôi:
```
FROM prom/prometheus:v2.30.3 AS prometheus-federtion
COPY prometheus-federtion.yml /etc/prometheus/prometheus.yml
FROM prom/prometheus:v2.30.3
COPY --from=prometheus-federtion /etc/prometheus/prometheus.yml /etc/prometheus/prometheus.yml
EXPOSE 9090
CMD ["--config.file=/etc/prometheus/prometheus.yml"]
```
