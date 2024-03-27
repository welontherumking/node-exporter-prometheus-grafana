Steps:

1. Create a `docker-compose.yml` and paste the configuration into this file
```
version: "3.9"

services:
  node-exporter:
    container_name: NodeExporter
    image: prom/node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude'
      - '^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)'

  prometheus:
    container_name: Prometheus
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - prom-data:/prometheus
      - prom-configs:/etc/prometheus

  grafana:
    container_name: Grafana
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - grafana-configs:/etc/grafana

volumes:
  grafana-data:
  grafana-configs:
  prom-data:
  prom-configs:
```

2. Start a stack of containers:
```
docker-compose up -d
```

3. Checking it out. Go to http://server-ip:3000/ and login to the admin account then Grafana ask you to change your password, change it

3. Adding Prometheus to Grafana:
- Click `Add your first data source`
- Select Prometheus
- Enter the address `http://Prometheus:9090` in the field `Prometheus server URL`
- Click `Save & test`

4. Export Grafana dashboard
- Go to https://grafana.com/grafana/dashboards/12486-node-exporter-full/ and download JSON
- Click `Create your first dashboard` in Grafana then `Import dashboard`
- Paste the copied content from the downloaded file and click `Load`
- Select our Prometheus and click `Import`

5. Add node-exporter in `prometheus.yml`
```
sudo nano /var/lib/docker/volumes/altlinux_prom-configs/_data/prometheus.yml
```

Add this to the end of the file

```
    - job_name: "node-exporter"
      static_configs:
        - targets: ["NodeExporter:9100"]
```

Restart containers
```
docker-compose restart
```

Go to the prometheus web interface via http://server-ip:9090 then go to `Status` -> `Targets` section. There should be a new Targets that we added in the `prometheus.yml` file

### Now in http://<server-ip>:3000 you can watch the graphs
