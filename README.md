# docker-exporter

for this configurations you can collect all docker container metrics from docker host with cadvisor.

### cadvisor insallation

you can run cadvisor on docker-compose file on docker host with host network. so prometheus directly collect metrics from host network and port.

```
version: "3.4"
services:
  cadvisor:
    container_name: cadvisor
    image: gcr.io/cadvisor/cadvisor:latest
    network_mode: "host"
    ports:
      - "8888:8080"
    volumes:
      - "/:/rootfs"
      - "/var/run:/var/run"
      - "/sys:/sys"
      - "/var/lib/docker/:/var/lib/docker"
      - "/dev/disk/:/dev/disk"
    privileged: true
```

### prometheus

then you can install prometheus binary and run prometheus as a service with /etc/prometheus/prometheus.yml

```
mkdir -p /etc/prometheus
touch /etc/prometheus/prometheus.yml
```

add below configuration on yml file.

```
global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['dockerhostip:8080']
```

you can add systemd file as prometheus service.

```
touch /etc/systemd/system/prometheus.service
```

add below configuration on service file.

```
[Unit]
Description=Prometheus
After=network.target

[Service]
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml
Restart=always

[Install]
WantedBy=default.target
```

then reload deamon and start service.

```
systemctl daemon-reload
systemctl start prometheus
```

### grafana

firstle need add data source for docker host

<img width="920" alt="image" src="https://github.com/alperen-selcuk/docker-exporter/assets/78741582/6267bc19-eafa-4131-a041-4d75b4dec86a">

then you can import this dashboard to grafana. id=14282

and you can collect all docker containers metric.



