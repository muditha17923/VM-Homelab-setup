# 🖥️ VM Monitoring Homelab — Prometheus + Grafana + Node Exporter

A beginner-friendly Linux monitoring project using **Alpine Linux, Ubuntu Server, Node Exporter, Prometheus, and Grafana**.

The goal of this project was to build a small monitoring system that collects resource metrics from an Alpine Linux VM and visualizes them through a Grafana dashboard.

## 🏗️ Architecture

```text
                  ┌──────────────────────────┐
                  │      Ubuntu Server VM    │
                  │                          │
                  │  Prometheus :9090       │
                  │  Grafana :3000           │
                  └────────────┬─────────────┘
                               │
                         Prometheus
                         scrapes metrics
                               │
                               ▼
                  ┌──────────────────────────┐
                  │       Alpine Linux VM    │
                  │                          │
                  │  Node Exporter :9100     │
                  └──────────────────────────┘
```

## 🧰 Technologies Used

- Alpine Linux
- Ubuntu Server
- Node Exporter
- Prometheus
- Grafana
- KVM/libvirt
- PromQL
- Linux networking

## ⚙️ Setup

### 1. Alpine Linux VM

Installed Node Exporter on the Alpine VM and exposed its metrics on:

```text
:9100
```

Verified the exporter with:

```bash
wget -qO- http://127.0.0.1:9100/metrics
```

The metrics endpoint returned system metrics such as CPU, memory, filesystem, and network statistics.

---

### 2. Prometheus

Prometheus was installed on the Ubuntu Server VM.

The Alpine VM was added as a Prometheus scrape target:

```yaml
scrape_configs:
  - job_name: "alpine"
    static_configs:
      - targets: ["ALPINE_VM_IP:9100"]
```

After restarting Prometheus, the target was verified through:

```text
http://UBUNTU_VM_IP:9090/targets
```

The Alpine target appeared as:

```text
UP
```

---

### 3. Grafana

Grafana was installed on the Ubuntu Server VM and configured to use Prometheus as its data source.

Prometheus URL:

```text
http://localhost:9090
```

After connecting the data source, Grafana was used to create a custom VM monitoring dashboard.

## 📊 Dashboard Panels

### CPU Usage

```promql
100 - (
  avg by (instance) (
    rate(node_cpu_seconds_total{
      job="alpine",
      mode="idle"
    }[5m])
  ) * 100
)
```

### Memory Usage

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes{job="alpine"}
  /
  node_memory_MemTotal_bytes{job="alpine"}
)
```

### Disk Usage

To monitor the main filesystem:

```promql
100 * (
  1 -
  node_filesystem_avail_bytes{
    job="alpine",
    mountpoint="/"
  }
  /
  node_filesystem_size_bytes{
    job="alpine",
    mountpoint="/"
  }
)
```

### Network Receive

```promql
rate(node_network_receive_bytes_total{
  job="alpine",
  device!="lo"
}[5m])
```

### Network Transmit

```promql
rate(node_network_transmit_bytes_total{
  job="alpine",
  device!="lo"
}[5m])
```

### VM Uptime

```promql
time() - node_boot_time_seconds{job="alpine"}
```

## 📈 Dashboard Layout

```text
┌────────────────┬────────────────┬────────────────┐
│   CPU Usage    │ Memory Usage   │   Disk Usage   │
├────────────────┴────────────────┴────────────────┤
│                  CPU Over Time                   │
├───────────────────────────────┬──────────────────┤
│      Network Receive          │ Network Transmit │
├───────────────────────────────┴──────────────────┤
│                  VM Uptime                       │
└──────────────────────────────────────────────────┘
```

## 🔍 Troubleshooting

A few useful checks during the setup:

### Check Node Exporter

```bash
curl http://ALPINE_VM_IP:9100/metrics
```

### Check Prometheus

```bash
sudo systemctl status prometheus
```

### Check Grafana

```bash
sudo systemctl status grafana-server
```

### Test Prometheus connectivity

```promql
up
```

Expected result:

```text
job="alpine"
instance="ALPINE_VM_IP:9100"
1
```

A value of `1` indicates that Prometheus can successfully scrape the Node Exporter.

## 🎯 What I Learned

Through this project I practiced:

- Linux VM administration
- KVM/libvirt networking
- Installing and configuring Node Exporter
- Prometheus scrape configuration
- PromQL queries
- Grafana dashboards
- Linux CPU, memory, disk and network monitoring
- Troubleshooting monitoring connectivity
- Understanding the relationship between exporters, Prometheus and Grafana

## 🚀 Future Improvements

Next steps for this homelab:

- [ ] Install Node Exporter on the Ubuntu VM
- [ ] Monitor both VMs from one dashboard
- [ ] Add Grafana alerts
- [ ] Create CPU, RAM and disk thresholds
- [ ] Add more Alpine/Debian VMs
- [ ] Monitor VM network traffic
- [ ] Add Docker/container monitoring
- [ ] Build a complete homelab monitoring stack

## 📚 Monitoring Stack

```text
Node Exporter
      ↓
 Prometheus
      ↓
   Grafana
      ↓
Dashboard
```

This project is part of my ongoing **Linux, Networking, DevOps and Homelab learning journey**.