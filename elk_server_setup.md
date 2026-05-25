# ELK Stack Setup Guide (ELK Server Only) - Ubuntu 24.04

## Architecture

App Server (Filebeat + Java App) ↓ Logstash :5044 ↓ Elasticsearch :9200
↓ Kibana :5601

------------------------------------------------------------------------

## EC2 Requirements

Instance: m7-flex.large or similar\
OS: Ubuntu 24.04\
Storage: Minimum 30GB
## Security Group Inbound Rules

| Port | Purpose                          | Source                        |
|------|----------------------------------|-------------------------------|
| 22   | SSH                              | My IP                         |
| 5044 | Logstash                         | App Server Security Group     |
| 5601 | Kibana                           | My IP                         |
| 9200 | Elasticsearch API *(optional)*   | My IP                         |

------------------------------------------------------------------------

## Update Server

``` bash
sudo apt update && sudo apt upgrade -y

sudo apt install curl gpg apt-transport-https unzip -y
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

## Install Elasticsearch + Logstash + Kibana

``` bash
sudo apt install elasticsearch logstash kibana -y
```

------------------------------------------------------------------------

## Elasticsearch Configuration

Edit:

``` bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Replace entire file:

``` yaml
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

------------------------------------------------------------------------

## Elasticsearch Heap

Create:

``` bash
sudo nano /etc/elasticsearch/jvm.options.d/heap.options
```

Add:

``` text
-Xms2g
-Xmx2g
```

------------------------------------------------------------------------

## Start Elasticsearch

``` bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl restart elasticsearch
```

Verify:

``` bash
curl http://localhost:9200
```

Or access using Public IP
```json
http://ELK_PUBLIC_IP:5601
```
Expected:
```json
{
  "name": "monitoring-node",
  "cluster_name": "elk-cluster"
}
```

------------------------------------------------------------------------

## Logstash Configuration

Create:

``` bash
sudo nano /etc/logstash/conf.d/logstash.conf
```

Add:

``` ruby
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

Test:

``` bash
sudo /usr/share/logstash/bin/logstash --config.test_and_exit -f /etc/logstash/conf.d/logstash.conf
```

Start:

``` bash
sudo systemctl enable logstash
sudo systemctl restart logstash
```

Verify:

``` bash
sudo ss -tulnp | grep 5044
```

Expected:

``` text
*:5044
```

------------------------------------------------------------------------

## Kibana Configuration

Edit:

``` bash
sudo nano /etc/kibana/kibana.yml
```

Replace with:

``` yaml
server.port: 5601
server.host: "0.0.0.0"
server.name: "elk-stack-server"

elasticsearch.hosts: ["http://localhost:9200"]

monitoring.ui.container.elasticsearch.enabled: true

logging:

  appenders:

    file:

      type: file
      fileName: /var/log/kibana/kibana.log

      layout:
        type: json

  root:

    appenders:
      - default
      - file

    level: info

pid.file: /run/kibana/kibana.pid

path.data: /var/lib/kibana
```

Start:

``` bash
sudo systemctl enable kibana
sudo systemctl restart kibana
```

Verify:

``` bash
sudo systemctl status kibana
```

Open:

``` text
http://ELK_PUBLIC_IP:5601
```

------------------------------------------------------------------------

## Verify ELK Stack

``` bash
curl http://localhost:9200

sudo systemctl status elasticsearch

sudo systemctl status logstash

sudo systemctl status kibana
```

Expected:

-   Elasticsearch active
-   Logstash active
-   Kibana active
-   Port 5044 listening
-   Port 9200 listening
-   Port 5601 accessible