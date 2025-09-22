---
title: "Building a Complete Monitoring & Alerting Stack with Prometheus, Grafana, and Alertmanager"
seoTitle: "Monitoring with Prometheus and Grafana"
seoDescription: "Use Prometheus, Grafana, and Alertmanager for real-time metrics, dashboards, and notifications in modern applications"
datePublished: Mon Sep 22 2025 05:49:05 GMT+0000 (Coordinated Universal Time)
cuid: cmfuphoib000002jrajok0ufu
slug: building-a-complete-monitoring-and-alerting-stack-with-prometheus-grafana-and-alertmanager
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1758263305117/b2c476e7-a726-4d47-bfa5-747aab6cd24b.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1758520374499/a1a73618-128f-4186-85fe-72cac09aa026.png
tags: cloud, docker, kubernetes, cloud-computing, devops, prometheus, grafana, devops-articles, grafana-monitoring

---

# Introduction

Modern applications are complex, distributed, and constantly evolving. Keeping them healthy requires more than just logs. It demands real time visibility, insightful dashboards, and proactive alerts. That is where a solid monitoring stack comes in. By combining **Prometheus** for metrics collection, **Grafana** for visualization, and **Alertmanager** for intelligent notifications, you can build a complete monitoring and alerting solution that helps you detect issues before they impact users.

In this guide we will walk through setting up Prometheus, Grafana, and Alertmanager step by step, explore how they integrate, and show you how to create dashboards and alerts that keep you in control of your systems. Whether you are running microservices in Kubernetes, scaling applications in the cloud, or managing on premises infrastructure, this stack provides the observability foundation you need.

In this guide, I'll walk you through how I built a complete, production-inspired monitoring stack from scratch that includes:

* **A sample web application (Node.js)** to monitor
    
* **Infrastructure metrics** (CPU, RAM, Disk, Network)
    
* **Docker container resource usage**
    
* **HTTP uptime monitoring**
    
* **Real-time email alerts** when things go wrong
    
* **Beautiful Grafana dashboards** (industry-standard IDs)
    
* All deployed via **Docker Compose** — portable, reproducible, modern
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758519355466/d465560a-a38e-4c3a-a84e-2a829cf329d0.png align="center")
    

### Prerequisites

You’ll need:

* **Docker + Docker Compose** installed
    
    * [Install Docker Desktop](https://docs.docker.com/desktop/) (Windows/Mac)
        
    * [Install Docker Engine](https://docs.docker.com/engine/install/) (Linux)
        

Verify with:

`docker --version`

`docker-compose --version`

# Step 1: Project Structure

First, create a project folder for our files. All configuration will live here, making the project portable and easy to share.

```bash
mkdir prometheus-grafana-alerts-stack
cd prometheus-grafana-alerts-stack
```

# Step 2: Create a Real App to Monitor (Node.js)

Monitoring nothing is pointless, so let's build a simple web app that we can use as a target for our monitoring stack.

**Create** `app.js`

This simple Node.js server listens on port 3001 and responds to all requests with "Hello from your monitored app! ".

```javascript

const http = require('http');

const PORT = 3001;

const server = http.createServer((req, res) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from your monitored app! \n');
});

server.listen(PORT, () => {
  console.log(`App running on http://localhost:${PORT}`);
});
EOF
```

**Create** `package.json`

This file defines our app's metadata and a simple `start` script.

Bash

```javascript

{
  "name": "monitored-app",
  "version": "1.0.0",
  "description": "Simple app to monitor with Prometheus + Grafana",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
```

## **Create** `Dockerfile`

The `Dockerfile` packages our Node.js app into a clean, reproducible container.

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3001

CMD ["npm", "start"]
```

`FROM node:18-alpine`: Starts with a small, lightweight Node.js base image.

`WORKDIR /app`: Sets the main working folder inside the container to `/app`.

`COPY package*.json ./`: Copies the dependency files first to enable caching.

`RUN npm install`: Installs all the project dependencies.

`COPY . .`: Copies the remaining application code.

`EXPOSE 3001`: Documents that the app listens on port 3001.

`CMD ["npm", "start"]`: Defines the command to run the application when the container starts.

## **Build & Run the App**

Now, build and run the app in a detached Docker container.

```dockerfile
docker build -t my-monitored-app .
docker run -d --name my-app -p 3001:3001 my-monitored-app
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758263788385/2509c935-e2e1-4aab-ba89-bf647dfade8a.png align="center")

**Test it:** Open `http://localhost:3001` in your browser. You should see: `Hello from your monitored app!`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758263909692/13a18729-5d10-49c3-96e4-eac6b064dd40.png align="center")

---

# Step 3: Create the Full Monitoring Stack with Docker Compose

Now for the main event! The `docker-compose.yml` file defines all the services in our monitoring stack, including Prometheus, Grafana, Alertmanager, and the various **exporters** that collect metrics.

* **Prometheus**: The heart of the stack, it scrapes metrics from all the exporters.
    
* **Grafana**: The visualization layer, it creates beautiful dashboards from the data stored in Prometheus.
    
* **Alertmanager**: Handles alerts from Prometheus, deduplicating, grouping, and sending them out (e.g., via email).
    
* **Node Exporter**: Collects host-level metrics like CPU, RAM, and disk usage.
    
* **cAdvisor**: Collects metrics about the resource usage of running Docker containers.
    
* **Blackbox Exporter**: Checks the availability of external endpoints, like our web app.
    

**Create** `docker-compose.yml`

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:v2.49.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert_rules.yml:/etc/prometheus/alert_rules.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--web.enable-lifecycle'
    depends_on:
      - alertmanager
    restart: unless-stopped

  grafana:
    image: grafana/grafana:10.3.1
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=Admin123!
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-storage:/var/lib/grafana
    depends_on:
      - prometheus
    restart: unless-stopped

  nodeexporter:
    image: prom/node-exporter:v1.7.0
    container_name: nodeexporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.47.2
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    privileged: true
    restart: unless-stopped

  blackbox:
    image: prom/blackbox-exporter:v0.24.0
    container_name: blackbox
    ports:
      - "9115:9115"
    volumes:
      - ./blackbox.yml:/etc/blackbox_exporter/config.yml
    command:
      - '--config.file=/etc/blackbox_exporter/config.yml'
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    restart: unless-stopped

volumes:
  grafana-storage:
```

---

### Service Breakdown

#### **Prometheus**

This is the **central monitoring server** that collects and stores metrics.

* `image`: Specifies the official Prometheus image from Docker Hub.
    
* `ports`: Exposes port `9090` on the host, allowing you to access the Prometheus UI.
    
* `volumes`: Mounts local configuration files (`prometheus.yml` and `alert_rules.yml`) into the container, so you can easily customize its behavior without rebuilding the image.
    
* `depends_on`: Ensures that the `alertmanager` container starts before Prometheus.
    
* `restart`: Automatically restarts the container if it stops, unless you manually stop it.
    

#### **Grafana**

This is the **visualization tool** that creates dashboards from the metrics collected by Prometheus.

* `image`: Specifies the official Grafana image.
    
* `ports`: Exposes port `3000` on the host for accessing the Grafana UI.
    
* `environment`: Sets environment variables to configure Grafana, such as the default admin password and disabling user sign-ups.
    
* `volumes`: Uses a named volume (`grafana-storage`) to persist Grafana data (dashboards, settings) even if the container is removed.
    
* `depends_on`: Ensures Prometheus starts first, as Grafana needs it as a data source.
    

#### **Exporters**

These are tools that **collect metrics** from different systems and expose them for Prometheus to scrape.

* `nodeexporter`: Scrapes **host-level metrics** like CPU, memory, and disk usage by mounting the host's `/proc`, `/sys`, and `/` directories as read-only volumes.
    
* `cadvisor`: Collects **Docker container metrics** (e.g., CPU and memory usage per container) by mounting Docker-related host directories. The `privileged: true` flag gives it the necessary permissions.
    
* `blackbox`: Checks if **network endpoints are available** (e.g., HTTP uptime). It is configured to run checks based on the `blackbox.yml` file.
    

#### **Alertmanager**

This service **manages alerts** sent by Prometheus.

* `image`: Specifies the official Alertmanager image.
    
* `ports`: Exposes port `9093` for the Alertmanager UI.
    
* `volumes`: Mounts the local `alertmanager.yml` file for configuration, which specifies how to send alerts (e.g., to an email address).
    

# Step 4: Configure Prometheus + Alert Rules

Prometheus needs to know what to monitor and when to fire an alert. We'll configure its scraping jobs and set up some useful rules.

**Create** `prometheus.yml`

This configuration file tells Prometheus where to find the various exporters, including our `my-app-monitor` job, which uses the Blackbox Exporter to check our app's uptime.

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['nodeexporter:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  - job_name: 'my-app-monitor'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - http://host.docker.internal:3001  # Mac/Windows
          # - http://172.17.0.1:3001         # Linux (if needed)
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox:9115
```

***Note:*** *For Linux, replace* `host.docker.internal` with your Docker host's IP. You can find this by running `ip addr show docker0`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758264190329/926e6ac7-047d-4ecc-9009-8a94fd9b66d1.png align="center")

### Explanation of `prometheus.yml`

```bash
global:
  scrape_interval: 15s
  evaluation_interval: 15s
```

* `global`: Defines default settings for Prometheus.
    
* `scrape_interval: 15s` → Prometheus collects metrics every 15 seconds.
    
* `evaluation_interval: 15s` → Prometheus evaluates alerting and recording rules every 15 seconds.
    

```bash
rule_files:
  - "alert_rules.yml"
```

* Prometheus loads alerting/recording rules from external files.
    
* Here, it loads `alert_rules.yml`, which contains conditions for triggering alerts.
    

```bash
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093
```

* Defines where Prometheus should send alerts.
    
* **Alertmanager** (running at `alertmanager:9093`) handles alert notifications (e.g., email, Slack, PagerDuty).
    

### Scrape Configurations (Data Sources)

This tells Prometheus what to monitor.

```bash
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

* **Job:** `prometheus` → Prometheus monitors itself on port `9090`.
    

```bash
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['nodeexporter:9100']
```

* **Job:** `node-exporter` → Collects host-level metrics (CPU, memory, disk, network) from Node Exporter running at `9100`.
    

```bash
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

* **Job:** `cadvisor` → Collects container-level metrics (CPU, memory, disk I/O, network) from cAdvisor on port `8080`.
    

```bash
  - job_name: 'my-app-monitor'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - http://host.docker.internal:3001  # Mac/Windows
          # - http://172.17.0.1:3001         # Linux (if needed)
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox:9115
```

* **Job:** `my-app-monitor` → Uses the **Blackbox Exporter** to probe an application.
    
* `metrics_path: /probe` → Endpoint used by Blackbox Exporter.
    
* `params.module: [http_2xx]` → Checks if the target app returns HTTP `2xx` (success) responses.
    
* `targets`: The application to monitor (example: [`http://host.docker.internal:3001`](http://host.docker.internal:3001)).
    
    * For Linux, you may need to replace it with [`http://172.17.0.1:3001`](http://172.17.0.1:3001).
        
* **Relabeling configs**:
    
    * Passes the target URL to the Blackbox Exporter.
        
    * Sets `instance` label to the probed target.
        
    * Redirects scraping to `blackbox:9115` (the Blackbox Exporter container).
        

**Summary:**  
This configuration tells Prometheus to:

1. Monitor itself (`prometheus:9090`).
    
2. Collect system metrics from Node Exporter.
    
3. Collect container metrics from cAdvisor.
    
4. Probe your custom application with the Blackbox Exporter to ensure it is reachable and healthy.
    
5. Send alerts to Alertmanager for processing.
    

## **Create** `alert_rules.yml`

These are the rules Prometheus will evaluate. If the conditions are met, an alert will fire. We're setting up four rules:

* `HighCpuUsage`
    
* `HighMemoryUsage`
    
* `MyAppDown`
    
* `ContainerRestarted`
    

```yaml
# alert_rules.yml
groups:
- name: system-alerts
  rules:
  - alert: HighCpuUsage
    expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is {{ $value }}% for more than 2 minutes."

  - alert: HighMemoryUsage
    expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High Memory usage on {{ $labels.instance }}"
      description: "Memory usage is {{ $value }}% for more than 2 minutes."

  - alert: MyAppDown
    expr: probe_success{job="my-app-monitor"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "My App is down"
      description: "HTTP probe failed for my-app. Service is unreachable."

  - alert: ContainerRestarted
    expr: changes(container_last_seen[5m]) > 0
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "Container restarted: {{ $labels.name }}"
      description: "Container {{ $labels.name }} was restarted in the last 5 minutes."
```

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758266508528/1a74457b-370b-4991-a2b4-b4e6ee37fe38.png align="center")

### Explanation of `alert_rules.yml`

This file defines **alerting rules** that Prometheus uses to detect problems in your infrastructure and applications.

### Structure:

* `groups`: Organizes related alerts together.
    
* `rules`: Each alert rule defines a condition, severity, and description.
    

### Alert Rules Breakdown

#### 1\. **High CPU Usage**

```bash
- alert: HighCpuUsage
  expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "High CPU usage on {{ $labels.instance }}"
    description: "CPU usage is {{ $value }}% for more than 2 minutes."
```

* **Condition**: If average CPU usage &gt; 80% for more than 2 minutes.
    
* **Metric used**: `node_cpu_seconds_total` (excluding idle time).
    
* **Severity**: `warning`.
    
* **Action**: Fires a warning alert with CPU usage percentage and the affected instance.
    

#### 2\. **High Memory Usage**

```bash
- alert: HighMemoryUsage
  expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "High Memory usage on {{ $labels.instance }}"
    description: "Memory usage is {{ $value }}% for more than 2 minutes."
```

* **Condition**: If memory usage exceeds 85% for more than 2 minutes.
    
* **Metric used**: Total memory minus available memory.
    
* **Severity**: `warning`.
    
* **Action**: Fires a memory alert with usage % and the affected node.
    

#### 3\. **Application Down**

```bash
- alert: MyAppDown
  expr: probe_success{job="my-app-monitor"} == 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "My App is down"
    description: "HTTP probe failed for my-app. Service is unreachable."
```

* **Condition**: Blackbox Exporter probe fails (`probe_success == 0`).
    
* **For**: More than 1 minute.
    
* **Severity**: `critical`.
    
* **Action**: Fires a critical alert when your app is unreachable.
    

#### 4\. **Container Restart**

```bash
- alert: ContainerRestarted
  expr: changes(container_last_seen[5m]) > 0
  for: 1m
  labels:
    severity: warning
  annotations:
    summary: "Container restarted: {{ $labels.name }}"
    description: "Container {{ $labels.name }} was restarted in the last 5 minutes."
```

* **Condition**: Detects if a container has restarted in the last 5 minutes.
    
* **Metric used**: `container_last_seen`.
    
* **Severity**: `warning`.
    
* **Action**: Fires a warning alert when any container restarts unexpectedly.
    

**Summary:**  
This alerting configuration helps you detect:

* High CPU load (`> 80%`).
    
* High memory usage (`> 85%`).
    
* Application downtime (via Blackbox probe).
    
* Unexpected container restarts.
    

It ensures that both **infrastructure** (CPU, memory, containers) and **applications** (availability via probes) are being monitored.

# Step 5: Configure Alertmanager (Email Alerts via Gmail)

Alertmanager needs to know where to send alerts. This configuration uses Gmail's SMTP server to send emails.

**Create** `alertmanager.yml`

***🚨 IMPORTANT:*** *You must replace the placeholder values with your own Gmail address and a 16-digit app password.*\*\*

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'your-email@gmail.com'          # ← REPLACE
  smtp_auth_username: 'your-email@gmail.com' # ← REPLACE
  smtp_auth_password: 'your-16-digit-app-password' # ← REPLACE
  smtp_require_tls: true

route:
  receiver: 'email-notifications'

receivers:
  - name: 'email-notifications'
    email_configs:
      - to: 'your-email@gmail.com'           # ← REPLACE
        send_resolved: true
```

**How to get a Gmail App Password:**

1. Go to [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).
    
2. Select **"Mail"** for the app and **"Other"** for the device.
    
3. Name it "Prometheus Alert" and click **Generate**.
    
4. Copy the 16-digit password and paste it into `smtp_auth_password`.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758266636922/a4819e98-3ae5-4b7f-a1d9-39bf85f4f27b.png align="center")
    

# Step 6: Configure Blackbox Exporter (HTTP Monitoring)

Blackbox Exporter is a powerful tool that checks endpoints and exposes the results as metrics for Prometheus. We'll set it up to check if our app is reachable and returns a `2xx` HTTP status code.

**Create** `blackbox.yml`

```yaml
# blackbox.yml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      method: GET
      preferred_ip_protocol: "ip4"
```

---

## Explanation of `blackbox.yml`

This file configures the **Blackbox Exporter**, which Prometheus uses to probe endpoints (HTTP, TCP, ICMP, DNS). It defines different **modules** (probe types).

---

### Module: `http_2xx`

```bash
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      method: GET
      preferred_ip_protocol: "ip4"
```

* `modules`: Defines probe configurations that Prometheus can use. Each module specifies how to check a target.
    

#### Breakdown:

* `http_2xx` → Module name. It checks if an HTTP endpoint responds with a `2xx` status (success).
    
* `prober: http` → Uses HTTP protocol to probe targets.
    
* `timeout: 5s` → Probe must complete within 5 seconds, otherwise it fails.
    
* `http.method: GET` → Sends an HTTP GET request.
    
* `preferred_ip_protocol: "ip4"` → Prefers IPv4 over IPv6 when probing targets.
    

---

**Summary:**  
This `blackbox.yml` tells Blackbox Exporter to perform **HTTP GET requests** against targets, expecting a `2xx` response (OK). It’s the module that your `prometheus.yml` references under the `my-app-monitor` job.

# Step 7: Start Everything

All our configuration is complete. It's time to bring the entire stack online with a single command.

```bash
docker-compose up -d
```

Wait about 30 seconds for all containers to start and for Prometheus to begin scraping.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758511801344/ca3768b0-3951-4fca-8fdc-62e63e82d958.png align="center")

---

# Step 8: Verify & Import Dashboards

Now, let's confirm everything is working and make our data beautiful with Grafana dashboards.

**Check Targets** Go to [http://localhost:9090/targets](https://www.google.com/search?q=http://localhost:9090/targets). All five targets should show a `UP` status.

* `prometheus`
    
* `node-exporter`
    
* `cadvisor`
    
* `blackbox`
    
* `my-app-monitor`
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758511732661/e5f9a073-4677-48ca-aadf-ce27f757abff.png align="center")

**Access Grafana** Navigate to [http://localhost:3000](https://www.google.com/search?q=http://localhost:3000). The default login is `admin` / `Admin123!`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758511896447/d6a148a6-3343-4a7a-8818-c89c92394752.png align="center")

**Import Dashboards** Grafana allows you to import community-made dashboards by ID. These provide rich, pre-built visualizations.

1. Click the `+` icon on the left sidebar and select `Import`.
    
2. Enter the following IDs one by one, selecting `Prometheus` as the data source for each.
    

* **1860** → Node Exporter Full (Host Metrics)
    
* **14282** → Docker & Containers
    
* **7587** → Blackbox Exporter (HTTP Probes)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758511996878/207b4bd4-6f30-4ec0-9dc8-4d990ba5dc79.png align="center")
    

After importing, you'll have beautiful, production-ready dashboards showing your host, container, and application metrics.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758512188490/917c17c1-d2dd-4d8e-b4ba-207f6db55ca7.png align="center")

---

# Step 9: Test Alerts (Force an Alert!)

The final test: let's intentionally break our app to see if the alerts work as expected.

**1\. Stop your app:**

```dockerfile
docker stop my-app
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758512315942/4045d4db-ea79-4764-b58d-8b9c1355a3f5.png align="center")

**2\. Wait 60 seconds.** Go to [http://localhost:9090/alerts](https://www.google.com/search?q=http://localhost:9090/alerts). You should see the `MyAppDown` alert in a **FIRING** state. Within a minute or two, you should also get an **email notification**! 🎉

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758514244307/e937b3aa-663e-4010-9579-653b0895fb3f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758514298393/1ff87cb8-f909-4673-a268-e7e87a0d70ce.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758514410478/b9f1ff17-45b3-4b5a-be3c-decc95318a00.png align="center")

**3\. Start it again:**

```bash
docker start my-app
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758513873320/73b2a55e-f09a-451f-a1ad-a3c5ee4232f1.png align="center")

Wait another 60 seconds. The alert will automatically resolve, and you'll receive a second email notifying you that the issue is fixed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758514530836/05f04430-625a-4ee9-9435-d6d409ea73ea.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758514616177/bcf151d9-77ff-4243-b2b8-2648bb95d176.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1758513993303/ef5da2e1-4558-4b78-8d98-c6a79cd66f69.png align="center")

## **Cleanup**

To stop and remove everything:

docker-compose down

docker rm -f my-app

docker rmi my-monitored-app

docker volume prune #removes grafana-storage if no longer needed

## Common Errors and How to Fix Them

**1\. Prometheus service not starting**

* **Cause:** Wrong configuration syntax in `prometheus.yml`.
    
* **Fix:** Run `promtool check config prometheus.yml` to validate the config before restarting.
    

**2\. Grafana cannot connect to Prometheus**

* **Cause:** The Prometheus service endpoint is wrong or not reachable.
    
* **Fix:** Double check the URL in Grafana’s data source settings. For Kubernetes, make sure the service name and namespace are correct.
    

**3\. “No data” showing in Grafana dashboards**

* **Cause:** Prometheus is scraping no targets or targets are down.
    
* **Fix:** Check `Status > Targets` in the Prometheus UI to see if your exporters or endpoints are up.
    

**4\. Alertmanager not receiving alerts**

* **Cause:** Prometheus is not configured to send alerts, or alert rules are invalid.
    
* **Fix:** Validate your `alerting_rules.yml` with `promtool check rules`. Make sure Prometheus has `alertmanagers` configured under `prometheus.yml`.
    

**5\. Alerts not being sent to email/Slack**

* **Cause:** Misconfigured receivers in Alertmanager (wrong SMTP credentials, wrong Slack webhook URL).
    
* **Fix:** Review the `alertmanager.yml` config. Test the webhook or SMTP connection separately.
    

**6\. High memory or CPU usage by Prometheus**

* **Cause:** Too many time series or very frequent scrape intervals.
    
* **Fix:** Increase scrape interval (for example from `15s` to `30s`) or reduce unnecessary metrics.
    

**7\. Permission issues in Kubernetes deployments**

* **Cause:** Missing RBAC roles for Prometheus or Grafana pods.
    
* **Fix:** Ensure your manifests include `ClusterRole` and `ClusterRoleBinding` with the right API access.
    

**8\. Grafana dashboards not persisting after pod restart**

* **Cause:** No persistent volume attached to Grafana.
    
* **Fix:** Configure a `PersistentVolumeClaim` so dashboards and settings survive pod restarts.