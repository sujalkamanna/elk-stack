# ELK Stack Log Monitoring System using Filebeat and Java Application

## Project Overview

This project demonstrates a centralized logging and monitoring solution using the ELK Stack (Elasticsearch, Logstash, Kibana) integrated with Filebeat for collecting application logs from a Java-based application running on AWS EC2 Ubuntu servers.

The system collects logs from an application server, transfers them to Logstash using Filebeat, stores and indexes them in Elasticsearch, and visualizes them through Kibana dashboards.

The objective of this project is to provide real-time log monitoring, centralized log management, and data visualization for efficient troubleshooting and analysis.

---

## System Architecture

```text
+------------------------------------------------+
|              App Server (Ubuntu)               |
|------------------------------------------------|
| Java Application                               |
| Filebeat                                       |
| /opt/app/app.log                               |
+----------------+-------------------------------+
                 |
                 | Port: 5044
                 |
                 v
+------------------------------------------------+
|              ELK Server (Ubuntu)               |
|------------------------------------------------|
| Logstash                                       |
| Elasticsearch                                  |
| Kibana                                         |
+------------------------------------------------+
                 |
                 |
                 v
+------------------------------------------------+
|              Kibana Dashboard                  |
|------------------------------------------------|
| Discover                                       |
| Pie Charts                                     |
| Line Charts                                    |
| Error Analysis                                 |
| Host Activity                                  |
+------------------------------------------------+
```

---

## Features

- Centralized log collection
- Real-time log monitoring
- Application log parsing using Grok patterns
- Dashboard visualization
- Error log analysis
- Host-based monitoring
- Time-series log analytics
- Easy scalability
- Filebeat-based lightweight log shipping

---

## Uses of ELK Stack

The ELK Stack is widely used for centralized logging, monitoring, and data analysis in modern applications and infrastructure environments. It helps organizations collect logs from different systems, process them, store them efficiently, and visualize them for better decision-making.

### 1. Centralized Log Management

ELK collects logs from multiple applications and servers into a single location.

Example:

- Application logs
- System logs
- Web server logs
- Database logs
- Security logs

Benefits:

- Easy access to all logs
- Simplified troubleshooting
- Better organization

---

### 2. Real-Time Monitoring

ELK provides real-time monitoring of application and server activities.

Example:

- Monitor application performance
- Detect server failures
- Track memory usage
- Observe CPU utilization

Benefits:

- Immediate issue detection
- Reduced downtime
- Faster response time

---

### 3. Error Detection and Troubleshooting

Logstash processes and filters logs while Kibana visualizes them for analysis.

Example:

```text
ERROR Database Connection Failed
WARN Memory Usage High
```

Benefits:

- Quick identification of errors
- Root cause analysis
- Faster debugging

---

### 4. Security Monitoring

ELK can collect and analyze security-related logs.

Examples:

- Failed login attempts
- Unauthorized access attempts
- Firewall logs
- User activity tracking

Benefits:

- Detect suspicious activities
- Improve system security
- Monitor threats

---

### 5. Application Performance Monitoring

Developers can monitor application behavior and performance metrics.

Example:

- Request processing time
- API response time
- Service health monitoring

Benefits:

- Improved performance
- Better user experience
- Resource optimization

---

### 6. Infrastructure Monitoring

ELK can monitor cloud resources and server environments.

Examples:

- AWS EC2 logs
- Docker container logs
- Kubernetes logs
- System performance metrics

Benefits:

- Infrastructure visibility
- Resource tracking
- Capacity planning

---

### 7. Business Data Analytics

ELK can analyze application-generated data to obtain business insights.

Examples:

- User activity patterns
- Website traffic analysis
- Customer behavior monitoring

Benefits:

- Data-driven decisions
- Trend analysis
- Business insights

---

### 8. DevOps and CI/CD Monitoring

ELK can integrate with DevOps tools to monitor deployments and pipelines.

Examples:

- Jenkins logs
- Docker logs
- Kubernetes events
- Deployment status

Benefits:

- Faster deployment analysis
- Continuous monitoring
- Improved reliability

---

## Why ELK Stack for this Project?

This project uses ELK Stack because it provides:

- Real-time log collection
- Centralized log storage
- Easy log analysis
- Powerful visualizations
- Scalability
- Faster troubleshooting
- Better application monitoring


---

## Technology Stack

### Cloud Platform

- AWS EC2

### Operating System

- Ubuntu 24.04

### Backend Application

- Java
- Maven

### Monitoring Stack

- Elasticsearch 8.x
- Logstash 8.x
- Kibana 8.x
- Filebeat 8.x

### Other Tools

- Git
- Linux
- Shell Commands

---

## Infrastructure Setup

Two Ubuntu EC2 instances were used:

### ELK Server

Services running:

- Elasticsearch
- Logstash
- Kibana

Minimum configuration:

```text
Instance Type: m7-flex.large
RAM: 8GB
vCPU: 2
Storage: 30GB
```

Required Ports:

| Port | Service |
|-------|----------|
| 22 | SSH |
| 5044 | Logstash |
| 5601 | Kibana |
| 9200 | Elasticsearch |

---

### Application Server

Services running:

- Java Application
- Filebeat

Minimum configuration:

```text
Instance Type: m7-flex.large
RAM: 8GB
vCPU: 2
Storage: 20GB
```

Required Ports:

| Port | Service |
|-------|----------|
| 22 | SSH |

---

## Project Workflow

### Step 1

Java application generates logs:

```text
INFO User Login
ERROR Database Connection Failed
WARN Memory Usage High
```

---

### Step 2

Logs are written to:

```text
/opt/app/app.log
```

---

### Step 3

Filebeat continuously monitors:

```yaml
paths:
 - /opt/app/app.log
```

---

### Step 4

Filebeat forwards logs to:

```text
Logstash : 5044
```

---

### Step 5

Logstash processes logs using Grok filters:

Example:

```ruby
filter {

 grok {

   match => {

     "message" =>
     "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}"

   }

 }

}
```

---

### Step 6

Elasticsearch stores logs in:

```text
application-logs-yyyy.MM.dd
```

Example:

```text
application-logs-2026.05.25
```

---

### Step 7

Kibana visualizes logs through:

- Discover
- Pie Charts
- Time Charts
- Data Tables
- Dashboards

---

## Elasticsearch Configuration

```yaml
cluster.name: elk-cluster

node.name: elk-stack-server

path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

network.host: 0.0.0.0
http.port: 9200

discovery.type: single-node

xpack.security.enabled: false

xpack.security.enrollment.enabled: false

xpack.security.http.ssl:
 enabled: false

xpack.security.transport.ssl:
 enabled: false
```

---

## Logstash Configuration

```ruby
input {

 beats {

   host => "0.0.0.0"

   port => 5044

 }

}

filter {

 grok {

   match => {

      "message" =>
      "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}"

   }

 }

}

output {

 elasticsearch {

    hosts => ["http://localhost:9200"]

    index => "application-logs-%{+YYYY.MM.dd}"

 }

 stdout {

    codec => rubydebug

 }

}
```

---

## Filebeat Configuration

```yaml
filebeat.inputs:

- type: filestream

  id: app-log-input

  enabled: true

  paths:

   - /opt/app/app.log

output.logstash:

 hosts: ["ELK_PRIVATE_IP:5044"]
```

---

## Dashboard Visualizations

The Kibana dashboard contains:

### 1. Log Level Distribution

Pie chart showing:

- INFO logs
- ERROR logs
- WARN logs

---

### 2. Logs Over Time

Line graph showing:

- Number of logs generated over time

---

### 3. Error Log Table

Displays:

- Error messages
- Host name
- Timestamp

---

### 4. Host Activity

Displays:

- Log count generated by each host

---

## Verification Commands

### Elasticsearch

```bash
curl http://localhost:9200
```

### Logstash

```bash
sudo systemctl status logstash
```

### Filebeat

```bash
sudo filebeat test output
```

### Kibana

```bash
sudo systemctl status kibana
```

### View Indices

```bash
curl -X GET "http://localhost:9200/_cat/indices?v"
```

---

## Expected Output

```text
INFO User Login
ERROR Database Failed
WARN Memory High
```

---

## Advantages

- Centralized monitoring
- Real-time log analysis
- Faster issue detection
- Improved troubleshooting
- Scalable architecture
- Better system visibility

---

## Future Enhancements

- Add Docker monitoring
- Integrate Kubernetes logs
- Configure alert notifications
- Add Prometheus and Grafana
- Implement authentication and SSL
- Deploy using Terraform

---

## Conclusion

This project successfully implements a centralized logging system using the ELK Stack and Filebeat. The system enables efficient collection, processing, storage, and visualization of application logs, improving system monitoring and troubleshooting capabilities.