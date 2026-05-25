# App Server Setup Guide (Ubuntu 24.04)

## Architecture

Java Application ↓ app.log ↓ Filebeat ↓ Logstash (ELK Server:5044)

------------------------------------------------------------------------

## EC2 Requirements

Instance: m7-flex.large or similar\
OS: Ubuntu 24.04\
Storage: Minimum 20GB

## Security Group Inbound Rules

| Port | Purpose | Source |
|------|---------|--------|
| 22   | SSH     | My IP  |

------------------------------------------------------------------------

## Update Server

``` bash
sudo apt update && sudo apt upgrade -y

sudo apt install curl gpg openjdk-17-jdk git maven -y
```

Verify:

``` bash
java -version

mvn -version
```

------------------------------------------------------------------------

## Add Elastic Repository

``` bash
sudo install -d -m 0755 /usr/share/keyrings

curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor --yes -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
```

------------------------------------------------------------------------

## Install Filebeat

``` bash
sudo apt install filebeat -y
```

------------------------------------------------------------------------

## Configure Filebeat

Edit:

``` bash
sudo nano /etc/filebeat/filebeat.yml
```

Delete entire file and replace with:

``` yaml
# ============================== Filebeat ======================================

filebeat.inputs:

- type: filestream
  id: app-log-input
  enabled: true

  paths:
    - /opt/app/app.log


filebeat.config.modules:

  path: ${path.config}/modules.d/*.yml

  reload.enabled: false


setup.template.settings:

  index.number_of_shards: 1


output.logstash:

  hosts: ["ELK_PRIVATE_IP:5044"]


processors:

  - add_host_metadata:
      when.not.contains.tags: forwarded

  - add_cloud_metadata: ~

  - add_docker_metadata: ~

  - add_kubernetes_metadata: ~


logging.level: info

path.data: /var/lib/filebeat

path.logs: /var/log/filebeat
```

Replace:

``` text
ELK_PRIVATE_IP
```

with actual ELK server private IP.

------------------------------------------------------------------------

## Start Filebeat

``` bash
sudo systemctl enable filebeat

sudo systemctl restart filebeat
```

Verify:

``` bash
sudo filebeat test config

sudo filebeat test output

sudo systemctl status filebeat
```

Expected:

``` text
connection... OK
Active: active (running)
```

------------------------------------------------------------------------

## Clone Java Project

``` bash
cd ~

git clone YOUR_GITHUB_REPO_URL

cd PROJECT_NAME
```

I've used

[Boardgame Listing WebApp](https://github.com/jaiswaladi246/Boardgame)
------------------------------------------------------------------------

## Build Project

Using Maven:

``` bash
mvn clean package -DskipTests
```

Or Maven Wrapper:

``` bash
chmod +x mvnw

./mvnw clean package -DskipTests
```

Expected:

``` text
BUILD SUCCESS
```

------------------------------------------------------------------------

## Create Log Directory

``` bash
sudo mkdir -p /opt/app
```

------------------------------------------------------------------------

## Run Java Application

``` bash
nohup java -jar target/*.jar > /opt/app/app.log 2>&1 &
```

Verify:

``` bash
ps -ef | grep java
```

Expected:

``` text
java -jar
```

------------------------------------------------------------------------

## Generate Test Logs

``` bash
echo "INFO User Login $(date)" >> /opt/app/app.log

echo "ERROR Database Failed $(date)" >> /opt/app/app.log

echo "WARN High Memory Usage $(date)" >> /opt/app/app.log
```

Verify:

``` bash
tail -f /opt/app/app.log
```

------------------------------------------------------------------------

## Verify Full Pipeline

``` bash
sudo filebeat test output

sudo systemctl status filebeat
```

Expected:

-   Connection OK
-   Filebeat running
-   Logs forwarded to ELK server