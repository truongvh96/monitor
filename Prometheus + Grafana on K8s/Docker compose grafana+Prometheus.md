```
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - 9090:9090
    restart: unless-stopped
    volumes:
      - ./prometheus:/etc/prometheus
      - prom_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - 3000:3000
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=grafana
    volumes:
      - grafana-data:/var/lib/grafana
  node_exporter:
    image: imageName
    container_name: node_exporter
    command: 
      - '--path.rootfs=/host'
    pid: host
    restart: unless-stopped
    volumes:
      - '/:host:ro,rslave'
volumes:
  prom_data:
    driver: local
  grafana-data:
    driver: local
```
