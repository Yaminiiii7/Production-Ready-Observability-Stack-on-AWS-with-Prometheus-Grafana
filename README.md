# 🚀 Prometheus Observability Stack on AWS


** Table of Contents**

1. Project Overview
2. Architecture
3. Project Structure
4. Phase 1: Infrastructure Provisioning
5. Phase 2: Deploy Monitoring Stack
6. Phase 3: Configuration & Validation
7. Testing & Troubleshooting

---

## 📌 Project Overview
A **production-ready observability stack** deployed on **AWS EC2** using **Terraform, Docker Compose, Prometheus, Grafana, Node Exporter, and Alertmanager**.

- Infrastructure provisioning using Terraform (IaC)
- Containerized services with Docker Compose
- Metrics collection with Prometheus + Node Exporter
- Dashboards with Grafana
- Alerting via Alertmanager (Email & Slack)

---

## 🏗️ Architecture
```

                ┌─────────────────────────────┐
                │          AWS Cloud          │
                │                             │
                │  ┌───────────────────────┐  │
                │  │   EC2 (Ubuntu)        │  │
                │  │                       │  │
                │  │  ┌───────────────┐    │  │
Metrics (9100) ───▶  │  Node Exporter │   │  │
                │  │  └───────────────┘    │  │
                │  │           │           │  │
                │  │           ▼           │  │
                │  │  ┌───────────────┐    │  │
                │  │  │ Prometheus    │◀───── Grafana (3000)
                │  │  │ (9090)        │    │  │
                │  │  └───────────────┘    │  │
                │  │           │           │  │
                │  │           ▼           │  │
                │  │  ┌───────────────┐    │  │
                │  │  │ Alertmanager  │    │  │
                │  │  │ (9093)        │    │  │
                │  │  └───────────────┘    │  │
                │  │      │        │       │  │
                │  │      ▼        ▼       │  │
                │  │   Slack     Email     │  │
                │  └───────────────────────┘  │
                └─────────────────────────────┘
```

Components:
- EC2 (Ubuntu)
- Prometheus
- Grafana
- Node Exporter
- Alertmanager

Ports:
- Prometheus: 9090
- Grafana: 3000
- Alertmanager: 9093
- Node Exporter: 9100

---

## 📂 Project Structure
```
.
prometheus-observability-stack/

│
├── prometheus/
│   ├── prometheus.yml            # Main Prometheus config
│   ├── alertrules.yml             # Alert rules (CPU, memory, disk, down)
│   └── targets.json              # Dynamic scrape targets
│
├── alertmanager/
│   └── alertmanager.yml          # Email & Slack alert routing
│
├── grafana/
│   ├── dashboards/               # Preloaded dashboards (JSON)
│   └── datasources/              # Prometheus datasource config
│
├── terraform-aws/
│   ├── modules/
│   │   ├── ec2/                  # EC2 instance module
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   ├── outputs.tf
│   │   │   └── userdata.sh
│   │   └── security-group/       # Security group module
│   │       ├── main.tf
│   │       ├── variables.tf
│   │       └── outputs.tf
│   │
│   ├── prometheus-stack/         # Root Terraform stack
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   │
│   └── vars/
|        └── ec2.tfvars           # Environment-specific values
│
├── README.md                     # Project documentation
├── LICENSE                       # Apache 2.0 license
├── docker-compose.yml            # Run the monitoring stack
├── Makefile                      # Automation (IP updates, config refresh)


```

---

## ⚙️ Prerequisites
- AWS Account
- Terraform >= 1.6
- Docker & Docker Compose
- Slack Webhook (optional)
- Gmail App Password (optional)
---

## 🚀 Deployment

1. Install AWS CLI, terraform
   ```
   aws configure
   ```

3. Create AWS Key pair
  Via AWS Console:
  EC2 → Key Pairs → Create key pair
  Name: EC2_Keypair (or your preferred name)
  Download and save the .pem file

## Phase 1: Infrastructure Provisioning
### Step 1: Clone the Repository
```
git clone https://github.com/ObservabilityStack-Prometheus-grafana.git
cd ObservabilityStack-Prometheus-grafana
```
### Step 2: Get VPC id and subnet id from AWS console and replace in ec2.tfvars. Also replace owner and key_name

### Step 3: 
```
cd terraform-aws/prometheus-stack/
terraform fmt
terraform validate
terraform plan --var-file=../vars/ec2.tfvars
terraform apply --var-file=../vars/ec2.tfvars

```
### Step 4: Connect to EC2 instance
```
ssh -i EC2_Keypair.pem ubuntu@<public IP of VM>
```

### Step 5: Verify User Data Execution
```
tail -50 /var/log/cloud-init-output.log
docker --version
docker-compose --version
```
---

## Phase 2: Deploy Monitoring Stack

### Step 1: Clone the Repository
```
#On the EC2 instance
git clone https://github.com/ObservabilityStack-Prometheus-grafana.git
cd ObservabilityStack-Prometheus-grafana
```

### Step 2: Makefile
The Makefile automates IP address updates:
```bash
make all
```

### Step 3: Deploy the Stack
```bash
sudo docker-compose up -d
```

### Step 4: Verify Deployment
```bash
sudo docker-compose up -d
sudo docker ps
sudo docker logs prometheus
sudo docker logs grafana
sudo docker logs node-exporter
sudo docker logs alert-manager
```
---

## Phase 3: Configuration & Validation

Access URLs
```
• Prometheus: `http://YOUR-EC2-IP:9090`
• Grafana: `http://YOUR-EC2-IP:3000`
• Alert Manager: `http://YOUR-EC2-IP:9093`
• Node Exporter: `http://YOUR-EC2-IP:9100/metrics`
```

### Step 1: Validate Prometheus

1. Open Prometheus UI:
   `http://YOUR-EC2-IP:9090`

2. Check Targets:
   - Go to Status → Targets
   - Should see Host job with endpoint `YOUR-EC2-IP:9100`
   - State should be UP

3. Check Alert Rules:
   - Go to Status → Rules
   - Should see 4 alert groups:
     - test – InstanceDown
     - HighCPUUsage
     - HighMemoryUsage
     - HighStorageUsage

4. Test PromQL Queries:

```promql

# CPU usage by mode
avg by (instance,mode) (irate(node_cpu_seconds_total{mode!='idle'}[1m]))

# Memory usage
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100

# Disk usage
(node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes * 100

# Network traffic
rate(node_network_receive_bytes_total[5m])
```
## Step 2: Configure Grafana

1. Access Grafana:
   `http://YOUR-ec2-IP:3000`
   - Username: admin
   - Password: admin
   - Change password when prompted

2. Add Prometheus Data Source:

   Connections → Data Sources → Add data source → Prometheus

   - URL: `http://prometheus:9090`
   - Access: Server (default)

   - Click "Save & Test" — should see green checkmark

3. Import Node Exporter Dashboard:
   - Dashboards → Import → Enter ID: 10180 → Load

   - Select Prometheus data source → Import

4. View Dashboard:
   - You should now see comprehensive system metrics:
     - CPU usage per core
     - Memory usage
     - Disk I/O
     - Network traffic
     - System load
     - Uptime
   
## Step 3: Configure Alert Manager

1. Edit Alert Manager Config (optional):

```bash
nano alertmanager/alertmanager.yml
```
2. Email Configuration:

```yaml
receivers:
- name: 'email-notifications'
  email_configs:
  - to: 'your-email@gmail.com'
    from: 'alerts@yourdomain.com'
    smarthost: 'smtp.gmail.com:587'
    auth_username: 'your-email@gmail.com'
    auth_password: 'your-app-password'
    send_resolved: true
```

3. Slack Configuration:

```yaml
- name: 'slack-notifications'
  slack_configs:
  - api_url: "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
    channel: '#alerts'
    send_resolved: true
```

4. Restart Alert Manager:

```bash
sudo docker-compose restart alertmanager
```

## Testing & Troubleshooting

### Test 1: High CPU Alert

```bash
# Simulate high CPU usage
openssl speed -multi $(nproc --all) &

# Check in Prometheus:
# 1. Go to Alerts tab
# 2. HighCpuUsage should turn PENDING → FIRING
# 3. After 1 minute, alert fires

# Stop the test
killall openssl
```

## Test 2: High Storage Alert

```bash
# Create a large file (16GB)
dd if=/dev/zero of=testfile_16GB bs=1M count=16384

# Check in Prometheus:
# HighStorageUsage alert should fire

# Clean up
rm testfile_16GB
```

### Test 3: Instance Down Alert

```bash
# Stop node exporter
sudo docker stop node-exportporter

# Check Prometheus – target should be DOWN
# InstanceDown alert fires after 1 minute

# Restart node exporter
sudo docker start node-exporter
```
---

By completing this project, you've learned:

✔ Infrastructure as Code – Terraform modules, variables, outputs  
✔ Cloud Computing – EC2, Security Groups, VPC  
✔ Containerization – Docker, Docker Compose  
✔ Monitoring – Prometheus, exporters, PromQL  
✔ Visualization – Grafana dashboards  
✔ Alerting – Alert rules, routing, notifications  
✔ Linux – User data scripts, swap management  
✔ Automation – Makefile for configuration update

---
## Cleanup


```bash
# On local machine
cd terraform-aws/prometheus-stack/
terraform destroy --var-file=../vars/ec2.tfvars
```
---
## Getting Help

1. Prometheus Docs: `https://prometheus.io/docs/`
2. Grafana Docs: `https://grafana.com/docs/`
3. Terraform Docs: `https://www.terraform.io/docs/`
4. Project Issues: `https://github.com/techiescamp/devops-projects/issues`


## 📜 License
Apache License 2.0
