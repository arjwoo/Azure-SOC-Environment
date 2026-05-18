# Azure-SOC-Environment
Cloud-based SOC Environment on Microsoft Azure using DVWA, ELK Stack, Filebeat, Docker, and attack simulation for centralized log analysis and threat monitoring.

## Overview

This project demonstrates the deployment of a cloud-based SOC (Security Operations Center) in Microsoft Azure using:

* DVWA (Damn Vulnerable Web Application)
* ELK Stack (Elasticsearch, Kibana)
* Filebeat
* Azure Virtual Network segmentation
* Load Balancer
* Jumpbox architecture
* Attack simulation and log analysis

The objective of the project was to simulate realistic attack traffic, centralize logs, and analyze security events using Kibana.

---

# Description of the Topology

The primary purpose of this network is to expose a load-balanced and monitored instance of DVWA (Damn Vulnerable Web Application) inside Microsoft Azure.

The environment was designed to simulate a realistic SOC and cloud security monitoring architecture using segmented networking, centralized logging, and attack telemetry analysis.

Load balancing ensures high availability by distributing traffic across multiple DVWA virtual machines. A Jumpbox/Bastion host serves as the secure gateway into the private Azure environment using SSH-based access.

The ELK Stack (Elasticsearch + Kibana) provides centralized logging and monitoring capabilities for observing vulnerable application activity, system logs, and attack telemetry.

Filebeat is used to monitor and forward logs from the DVWA servers to Elasticsearch for indexing and analysis in Kibana.

The topology demonstrates concepts including:

* Azure virtual networking
* Network segmentation
* Bastion/jumpbox administration
* Load balancing
* Centralized logging
* SIEM-style monitoring
* Attack simulation and threat hunting
* Dockerized vulnerable infrastructure

---

# Architecture

## Components

| Component           | Purpose                               |
| ------------------- | ------------------------------------- |
| Jumpbox VM          | Secure administrative access          |
| DVWA VM 1           | Vulnerable web application node       |
| DVWA VM 2           | Vulnerable web application node       |
| ELK VM              | Centralized logging and visualization |
| Filebeat            | Log forwarding agent                  |
| Azure Load Balancer | Traffic distribution                  |

---

# Azure Network Design

## Virtual Network

```text
vnet-soc-lab
```

## Subnets

| Subnet            | Address Space | Purpose   |
| ----------------- | ------------- | --------- |
| frontend-subnet   | 10.0.1.0/24   | DVWA VMs  |
| backend-subnet    | 10.0.2.0/24   | ELK Stack |
| management-subnet | 10.0.3.0/24   | Jumpbox   |

---

# Security Architecture

## NSG Rules

### Frontend Subnet

* Allow HTTP (80) from Load Balancer
* Allow SSH (22) only from management subnet

### Backend Subnet

* Allow Elasticsearch (9200) from frontend subnet
* Allow Kibana (5601) from management subnet
* Allow SSH (22) from management subnet

### Management Subnet

* Allow SSH from administrator public IP only

---

# Infrastructure Deployment

## Jumpbox

* Ubuntu Server 22.04
* Public IP enabled
* SSH key authentication

## DVWA Deployment

Docker container deployed using:

```bash
sudo docker run -d -p 80:80 vulnerables/web-dvwa
```

## ELK Deployment

### Elasticsearch

```bash
sudo docker run -d \
--name elasticsearch \
--net elk \
-p 9200:9200 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms1g -Xmx1g" \
docker.elastic.co/elasticsearch/elasticsearch:7.10.2
```

### Kibana

```bash
sudo docker run -d \
--name kibana \
--net elk \
-p 5601:5601 \
-e ELASTICSEARCH_HOSTS=http://elasticsearch:9200 \
docker.elastic.co/kibana/kibana:7.10.2
```

---

# Secure Kibana Access

Kibana was accessed securely using SSH tunneling through the Jumpbox.

```bash
ssh -i mykey.pem -L 5601:10.0.2.4:5601 azureuser@<jumpbox-public-ip>
```

Browser Access:

```text
http://localhost:5601
```

---

# Filebeat Configuration

## Apache Module Enabled

```bash
sudo filebeat modules enable apache
```

## Docker Container Log Collection

```yaml
filebeat.inputs:
- type: container
  paths:
    - /var/lib/docker/containers/*/*.log
```

## Elasticsearch Output

```yaml
output.elasticsearch:
  hosts: ["http://10.0.2.4:9200"]
```

---

# Attack Simulations

## Brute Force

* Repeated login attempts against DVWA
* Observed repeated POST requests and HTTP 302 responses

## SQL Injection

Payload example:

```sql
' OR 1=1#
```

## Cross-Site Scripting (XSS)

Payload example:

```html
<script>alert('xss')</script>
```

## Command Injection

Payload example:

```bash
127.0.0.1 && whoami
```

## Network Scanning

```bash
nmap <target-ip>
```

---

# SOC Monitoring Workflow

```text
DVWA Attacks
      ↓
Docker Logs
      ↓
Filebeat
      ↓
Elasticsearch
      ↓
Kibana Dashboards
```

---

# Key Skills Demonstrated

* Azure networking
* Network segmentation
* NSG configuration
* Docker containerization
* ELK Stack deployment
* Filebeat configuration
* SSH tunneling
* Security monitoring
* SIEM fundamentals
* Attack simulation
* Log analysis
* Threat hunting

---

# Lessons Learned

* Importance of private network segmentation
* Centralized logging architecture
* Difference between host logs and container logs
* Secure access design using Jumpboxes
* Troubleshooting Elasticsearch container issues
* Understanding SOC telemetry pipelines

---

# Future Improvements

* Microsoft Sentinel integration
* Wazuh integration
* Suricata IDS
* Terraform automation
* Ansible automation
* Custom Kibana dashboards
* Alert rules and detections
* Threat intelligence feeds

# Conclusion

This SOC lab project demonstrates practical cloud security engineering skills by integrating vulnerable applications, centralized logging, attack simulation, and SIEM-based monitoring within Microsoft Azure.

