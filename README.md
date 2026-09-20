# -Dockerized-Zabbix-Monitoring-Stack
A complete monitoring environment built with Zabbix, Prometheus, Grafana, Node Exporter, and cAdvisor using Docker and Docker Compose.  The goal of this project is to create a centralized monitoring stack capable of monitoring both host-level system metrics and Docker container performance, while providing visualization through Grafana.
# Zabbix Monitoring Stack with Docker

A complete monitoring environment built with **Zabbix, Prometheus, Grafana, Node Exporter, and cAdvisor** using Docker and Docker Compose.

## 📌 Project Overview

The goal of this project was to build a centralized monitoring environment for monitoring both **Linux host resources** and **Docker containers**.

The monitoring stack consists of:

* Zabbix
* Prometheus
* Grafana
* Node Exporter
* cAdvisor
* Zabbix Agent
* Docker & Docker Compose

---

## 🏗️ Architecture

```text
                    ┌─────────────────┐
                    │     Grafana     │
                    │      :3000      │
                    └────────┬────────┘
                             │
                  ┌──────────┴──────────┐
                  │                     │
          ┌───────▼───────┐     ┌──────▼──────┐
          │     Zabbix    │     │ Prometheus  │
          │     Server    │     │    :9090    │
          │     :10051    │     └──────┬──────┘
          └───────┬───────┘            │
                  │              ┌─────┴─────────┐
                  │              │               │
          ┌───────▼───────┐ ┌────▼──────┐ ┌─────▼─────┐
          │ Zabbix Agent  │ │   Node    │ │  cAdvisor │
          │    :10050     │ │ Exporter  │ │   :8081   │
          └───────────────┘ │   :9100   │ └───────────┘
                            └───────────┘
```

---

## 🚀 Technologies

| Technology     | Purpose                   |
| -------------- | ------------------------- |
| Zabbix         | Infrastructure monitoring |
| Prometheus     | Metrics collection        |
| Grafana        | Data visualization        |
| Node Exporter  | Linux host metrics        |
| cAdvisor       | Docker container metrics  |
| Docker         | Containerization          |
| Docker Compose | Service orchestration     |
| Linux          | Host operating system     |

---

# 🐳 Docker Compose

The monitoring environment was deployed using Docker Compose.

Create the Docker Compose file:

```bash
nano docker-compose.yml
```

Start the stack:

```bash
docker compose up -d
```

Check running containers:

```bash
docker ps
```

The environment contains services such as:

* Zabbix Server
* Zabbix Web
* Database
* Prometheus
* Grafana

---

# 📊 Zabbix Agent

A Zabbix Agent was installed directly on the Linux host to collect local system metrics.

## Installation

```bash
sudo apt update
sudo apt install zabbix-agent -y
```

Configure the agent:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Set the following parameters:

```text
Server=<ZABBIX_SERVER_IP>
ServerActive=<ZABBIX_SERVER_IP>
Hostname=<HOSTNAME>
```

Enable and start the service:

```bash
sudo systemctl enable zabbix-agent
sudo systemctl start zabbix-agent
```

Check the service:

```bash
systemctl status zabbix-agent
```

---

# 🔗 Zabbix Agent with Dockerized Zabbix Server

Since the Zabbix Server is running inside Docker, the internal Docker IP was used for communication between the Agent and Zabbix Server.

The container IP can be checked using:

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' zabbix-server
```

Example:

```text
172.19.0.4
```

The Zabbix Agent configuration was then updated:

```text
Server=172.19.0.4
ServerActive=172.19.0.4
Hostname=<HOSTNAME>
```

Restart the Agent:

```bash
sudo systemctl restart zabbix-agent
```

The Zabbix host was then created from:

```text
Data collection → Hosts → Create host
```

The Agent interface was configured using the host machine IP and port `10050`.

---

# 🐳 cAdvisor

cAdvisor was deployed to monitor Docker containers and collect container-level metrics.

```bash
docker run -d --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/sys/fs/cgroup:/sys/fs/cgroup:ro \
  --publish=8081:8080 \
  google/cadvisor:latest \
  --housekeeping_interval=10s \
  --enable_load_reader=false \
  --containerd=/run/containerd/containerd.sock
```

cAdvisor was exposed on:

```text
http://<SERVER_IP>:8081
```

It provides container-level information such as:

* CPU usage
* Memory usage
* Container statistics
* Filesystem usage
* Container performance metrics

---

# 📈 Prometheus

Prometheus was deployed as part of the Docker Compose stack.

Example configuration:

```yaml
prometheus:
  image: ubuntu/prometheus:3-24.04_stable
  container_name: prometheus-container
  ports:
    - "9090:9090"
  volumes:
    - ./prometheus:/etc/prometheus
  depends_on:
    - zabbix-server
    - grafana
  networks:
    - zabbix-net
```

Start Prometheus:

```bash
docker compose up -d prometheus
```

Check the container:

```bash
docker ps
```

Prometheus Web UI:

```text
http://<SERVER_IP>:9090
```

---

# 🖥️ Node Exporter

Node Exporter was installed directly on the Linux host to collect system-level metrics.

Download and extract Node Exporter:

```bash
cd /opt

wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.linux-amd64.tar.gz

tar xvfz node_exporter-*.linux-amd64.tar.gz

cd node_exporter-*.linux-amd64
```

Run Node Exporter:

```bash
./node_exporter &
```

A systemd service was also configured so Node Exporter starts automatically with the operating system.

Node Exporter exposes metrics on:

```text
http://<HOST_IP>:9100/metrics
```

---

# ⚙️ Prometheus Configuration

Prometheus was configured to collect metrics from different monitoring components.

Example `prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'zabbix'
    static_configs:
      - targets: ['<ZABBIX_IP>:10051']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<HOST_IP>:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['<HOST_IP>:8081']
```

Restart Prometheus:

```bash
docker restart prometheus-container
```

---

# ✅ Prometheus Target Verification

Open:

```text
Prometheus → Status → Targets
```

The configured targets should appear as:

```text
prometheus       UP
zabbix           UP
node_exporter    UP
cadvisor         UP
```

This confirms that Prometheus is successfully collecting metrics from the configured monitoring endpoints.

---

# 📊 Grafana

Grafana was used as the visualization layer for the monitoring environment.

Grafana Web UI:

```text
http://<SERVER_IP>:3000
```

---

# 🔌 Grafana Zabbix Plugin

The **Zabbix plugin by Alexander Zobnin** was installed to integrate Zabbix with Grafana.

The plugin can be downloaded using:

```bash
wget https://grafana.com/api/plugins/alexanderzobnin-zabbix-app/versions/latest/download \
-O alexanderzobnin-zabbix-app.zip
```

The plugin was then extracted into the Grafana plugin directory and Grafana was restarted.

---

# 🔗 Grafana Data Sources

Two main data sources were configured in Grafana.

## Zabbix

```text
Connections → Data sources → Add data source → Zabbix
```

Example URL:

```text
http://<ZABBIX_SERVER_IP>/api_jsonrpc.php
```

## Prometheus

```text
Connections → Data sources → Add data source → Prometheus
```

Example URL:

```text
http://<PROMETHEUS_SERVER_IP>:9090
```

Both data sources were tested successfully from Grafana.

---

# 📋 Grafana Dashboard

A prebuilt Grafana dashboard was imported using dashboard ID:

```text
893
```

The dashboard provides an overview of system and container metrics.

---

# 🔄 Monitoring Flow

```text
                    Linux Host
                        │
          ┌─────────────┼──────────────┐
          │             │              │
          ▼             ▼              ▼
     Zabbix Agent   Node Exporter   cAdvisor
          │             │              │
          ▼             └──────┬───────┘
     Zabbix Server             │
          │                    ▼
          │                Prometheus
          │                    │
          └──────────┬─────────┘
                     ▼
                  Grafana
```

---

# 🎯 Project Result

The final environment provides centralized monitoring for:

* Linux host resources
* Docker containers
* CPU utilization
* Memory utilization
* Disk and filesystem metrics
* Network metrics
* Container performance
* Zabbix monitoring data
* Prometheus metrics

The monitoring data can be visualized through Grafana dashboards.

---

# 🧠 Skills Demonstrated

This project demonstrates hands-on experience with:

* Docker
* Docker Compose
* Zabbix
* Zabbix Agent
* Prometheus
* Grafana
* Node Exporter
* cAdvisor
* Linux
* systemd
* Container networking
* Monitoring architecture
* Infrastructure monitoring
* Metrics collection
* Dashboard configuration

---

# 🔮 Future Improvements

Planned improvements for this project:

* Alerting and notification integration
* SMS and messaging notifications
* Infrastructure monitoring automation with Ansible
* Monitoring as Code
* Custom Grafana dashboards
* Additional Prometheus exporters
* Automated deployment
* AI-assisted monitoring and anomaly detection

---

# 👩‍💻 Author

**Sevda Parhamrad**

Monitoring Specialist | Electronics & Mechatronics Engineer

**Skills:**

`Zabbix` `Grafana` `Prometheus` `Docker` `Linux` `Monitoring` `DevOps`

