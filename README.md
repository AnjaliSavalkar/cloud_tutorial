# 📊 Monitoring Stack with Grafana, Prometheus & Node Exporter (Docker)

This guide explains how to set up **Grafana**, **Prometheus**, and **Node Exporter** using **Docker** on a Windows or Linux machine for system monitoring and visualization.

---

## 🧱 Prerequisites

- Docker installed: [https://www.docker.com/get-started](https://www.docker.com/get-started)
- Basic terminal or command prompt knowledge
- Internet access to pull images

---

## 🚀 Setup Steps

### 🧰 Step 1: Create a Docker Network

Create a common network so containers can communicate.

```cmd
docker network create monitor
```

---

### 📥 Step 2: Pull Docker Images

```cmd
docker pull grafana/grafana
docker pull bitnami/prometheus
docker pull prom/node-exporter
```

---

### 🗂️ Step 3: Create Prometheus Configuration

Create this folder structure on your system:

```
monitoring/
└── prometheus/
    └── prometheus.yml
```

#### Sample `prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
```

Save this file as:  
`monitoring/prometheus/prometheus.yml`

---

### 🐳 Step 4: Run Docker Containers

> ⚠️ Run all from the root `monitoring` directory in **Command Prompt (CMD)**

#### 🔹 Prometheus

```cmd
docker run -d ^
  --name prometheus ^
  --network=monitor ^
  -p 9090:9090 ^
  -v %cd%\prometheus\prometheus.yml:/etc/prometheus/prometheus.yml ^
  bitnami/prometheus
```

#### 🔹 Grafana

```cmd
docker run -d ^
  --name=grafana ^
  --network=monitor ^
  -p 3000:3000 ^
  grafana/grafana
```

#### 🔹 Node Exporter

```cmd
docker run -d ^
  --name=node_exporter ^
  --network=monitor ^
  -p 9100:9100 ^
  prom/node-exporter
```

---

## ✅ Step 5: Verify Services

- Prometheus: [http://localhost:9090](http://localhost:9090)
- Grafana: [http://localhost:3000](http://localhost:3000)

---

## 📊 Grafana Setup

### 🔐 Login

- **Username:** `admin`
- **Password:** `admin` (change if prompted)

---

### 🔌 Add Prometheus Data Source

1. Navigate to: **Gear Icon → Data Sources → Add Data Source**
2. Select **Prometheus**
3. Set the URL to:

```
http://prometheus:9090
```

4. Click **Save & Test**
5. ✅ It should return: “Successfully queried the Prometheus API”

---

### 📈 Create CPU Utilization Panel

Use this query to view CPU usage:

```prometheus
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

1. Go to **+ → Dashboard → Add Panel**
2. Paste the above query
3. Set panel type to `Time series`
4. Click **Apply**

---

### 📦 Optional: Import Prebuilt Dashboard

1. Go to **Dashboards → Import**
2. Enter **Dashboard ID**: `1860`
3. Click **Load**
4. Select the Prometheus data source
5. Click **Import**

This imports a full Node Exporter dashboard.

---

## 🛑 Stopping Containers

```cmd
docker stop grafana prometheus node_exporter
docker rm grafana prometheus node_exporter
```

---

## 📌 Notes

- Always ensure all containers are on the **same Docker network** (`monitor`)
- Do **not** use `localhost` in Grafana data source — use container name (`prometheus`)
- Use **Windows CMD** (`%cd%`) instead of `$(pwd)` for volume paths

---

## 📚 References

- [Grafana Docker Hub](https://hub.docker.com/r/grafana/grafana)
- [Bitnami Prometheus Docker](https://hub.docker.com/r/bitnami/prometheus)
- [Node Exporter Docker](https://hub.docker.com/r/prom/node-exporter)
- [Grafana Dashboard 1860](https://grafana.com/grafana/dashboards/1860)

---

## 🙌 Author

This setup was documented for easy plug-and-play Grafana & Prometheus monitoring on Docker.  
Contributions and suggestions welcome!
