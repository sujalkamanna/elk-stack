# Kibana Data View and Dashboard Configuration Guide

---

## Step 1: Open Kibana

Open browser:

http://ELK_PUBLIC_IP:5601

---

## Step 2: Create Data View

### Navigate

☰ **Menu** → **Stack Management** → **Data Views** → **Create Data View**

## Fill in the Details

| Field | Value |
|------|--------|
| **Name** | `Boardgame Logs` |
| **Index pattern** | `application-logs-*` |
| **Timestamp field** | `@timestamp` |

## Final Step

Click **Save Data View**

---

## Step 3: Verify Data View

### Navigate

☰ **Menu** → **Discover**

## Select

`Boardgame Logs`

## Change Time Range

Top-right → **Last 15 minutes**

Change to:

- **Last 24 hours**
- **Last 7 days**

## Refresh Logs

Click **Refresh**

## Expected Logs

```text
INFO User Login
ERROR Database Failed
WARN High Memory Usage
````

---

## Step 4: Add Useful Columns

In **Discover** (left sidebar), search and add:

* `message`
* `host.name`
* `log.file.path`
* `timestamp`
* `level`

Click **+** beside each field to add it as a column.

---

## Step 5: Create Dashboard

Navigate:

☰ **Menu** → **Analytics** → **Dashboard** → **Create Dashboard**

Click:

**Create Visualization**

---

## Visualization 1: Log Level Distribution (Pie Chart)

* Type: Pie Chart
* Data View: `Boardgame Logs`
* Metric: Count
* Slice By: Terms → `level.keyword`

Save: **Log Level Distribution**

---

## Visualization 2: Logs Over Time

* Type: Line Chart
* X-axis: `@timestamp`
* Y-axis: Count

Save: **Logs Over Time**

---

## Visualization 3: Error Logs Table

* Type: Data Table
* Metric: Count
* Filter: `message : ERROR`
* Columns:

  * message
  * host.name
  * timestamp

Save: **Error Logs Table**

---

## Visualization 4: Host Activity

* Type: Bar Chart
* X-axis: host.name
* Y-axis: Count

Save: **Host Activity**

---

## Step 6: Build Dashboard

Add visualizations:

* Log Level Distribution
* Logs Over Time
* Error Logs Table
* Host Activity

Arrange layout and click **Save**

Dashboard Name:

**Boardgame Monitoring Dashboard**

---

## Step 7: Live Dashboard Testing

Run on App Server:

```bash
echo "INFO User Login $(date)" >> /opt/app/app.log
echo "ERROR Payment Failed $(date)" >> /opt/app/app.log
echo "WARN Memory High $(date)" >> /opt/app/app.log
```

Open dashboard → Click **Refresh**

---

## Expected Flow

```text
Java Application
↓
app.log
↓
Filebeat
↓
Logstash
↓
Elasticsearch
↓
Kibana Data View
↓
Dashboard Visualizations
```