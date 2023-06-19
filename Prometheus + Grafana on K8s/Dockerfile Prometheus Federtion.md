Để build image với config ta cần tạo 1 file config ``` prometheus-federtion.yaml ```
```
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
          - '<prometheus-server1>:9090'
          - '<prometheus-server1>:9090'
```
Sau khi đã có file config tạo dockerfile thôi:
```
FROM prom/prometheus:v2.30.3 AS prometheus-federtion
COPY prometheus-federtion.yaml /etc/prometheus/prometheus.yml
FROM prom/prometheus:v2.30.3
COPY --from=prometheus-federtion /etc/prometheus/prometheus.yml /etc/prometheus/prometheus.yml
EXPOSE 9090
CMD ["--config.file=/etc/prometheus/prometheus.yml"]
```
