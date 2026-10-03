# Centralized Log Aggregation and Monitoring using ELK Stack

## 1. Project Overview

This project implements a centralized log aggregation and monitoring solution using the **ELK Stack (Elasticsearch, Logstash, and Kibana)** with **Filebeat**.

The objective is to collect Nginx logs from multiple application servers, forward the logs to a centralized ELK server, parse and structure the log data, store the processed logs in Elasticsearch, and visualize important metrics in Kibana.

The project uses two application servers and one centralized ELK server. Nginx generates the application/server logs, Filebeat collects and forwards them, Logstash processes the events, Elasticsearch stores and indexes them, and Kibana provides dashboards and alert-rule configuration for monitoring.

---

## 2. Project Objectives

- Collect Nginx logs from multiple application servers.
- Forward logs centrally using Filebeat.
- Process and parse logs using Logstash.
- Store structured log data in Elasticsearch.
- Visualize log metrics using Kibana.
- Monitor HTTP error responses such as 4xx and 5xx.
- Configure an alert rule for abnormal HTTP error activity.
- Provide a centralized view of application-server logs.

---

## 3. Architecture
![src1](./img/ChatGPT%20Image%20Oct%204,%202026,%2012_21_41%20AM.png)
## 4. Technologies Used
	
* AWS EC2	
* Nginx	
* Filebeat	
* Logstash	
* Elasticsearch	
* Kibana	
* AWS Security Groups	
## 5. Infrastructure Setup

__Three EC2 instances were used.__

* __Application Server 1__
```
Ubuntu EC2 instance
Nginx installed
Filebeat installed
Nginx access logs collected from:
/var/log/nginx/access.log
```
* __Application Server 2__

`Same conffigration as Application Server 1`
* __ELK Server__

### The centralized server contains:
```
Elasticsearch
Logstash
Kibana
```
__Main ports:__
```
5044  → Logstash
5601  → Kibana
9200  → Elasticsearch
```
## 6. Nginx Configuration

Nginx was installed on both application servers.

Nginx generates access logs whenever a request is received.

Example:
```
127.0.0.1 - - [03/Oct/2026:12:00:00 +0000] "GET / HTTP/1.1" 200 615 "-" "curl"
```
The HTTP status code is important for monitoring.

Examples:
```
200 → Successful request
404 → Client error
500 → Server error
```
The project specifically monitors 4xx and 5xx HTTP errors.

__Generating a test 404 error__

A non-existing URL was requested:
```
curl http://localhost/THIS_IS_A_TEST_404
```
This generated a `404` entry in the Nginx access log.

The generated log was later processed and indexed into Elasticsearch.


## 7. Filebeat Configuration

Filebeat was installed on both application servers.

Its responsibility is to collect Nginx logs and send them to the centralized Logstash server.

The Logstash output was configured with:
```
output.logstash:
  hosts: ["ELK-SERVER-IP:5044"]
```
The Nginx module was enabled to collect Nginx logs.

Filebeat configuration was tested using:
```
sudo filebeat test config
```
The configuration was then checked and the Filebeat service was started.
```
sudo systemctl enable filebeat
sudo systemctl start filebeat
sudo systemctl status filebeat
```

## 8. Logstash Configuration

Logstash was configured on the ELK server to receive logs from Filebeat.

The Beats input listens on port:

`5044`

The pipeline was configured to:

1. Receive logs from Filebeat.
2. Parse Nginx access logs.
3. Extract important fields.
4. Identify HTTP status codes.
5. Classify HTTP errors.
6. Send structured data to Elasticsearch.

Important fields include:

`client_ip`

`method`

`request`

`status`

`bytes`

The configuration was stored under:

`/etc/logstash/conf.d/nginx.conf`

The configuration was tested using:
```
sudo -u logstash /usr/share/logstash/bin/logstash \
-f /etc/logstash/conf.d/nginx.conf \
--config.test_and_exit \
--path.settings /etc/logstash
```
The final validation returned:
```
Configuration OK
Config Validation Result: OK. Exiting Logstash
```

## 9. Log Parsing and Error Classification

Logstash parses the Nginx access log using a Grok pattern.

The important HTTP status field is extracted from the log.

The project classifies HTTP responses into:
```
4xx → Client errors
5xx → Server errors
```
__The processed data is sent to Elasticsearch using an index pattern similar to:__

`nginx-logs-YYYY.MM.dd`

Example index:

`nginx-logs-2026.10.03`
## 10. Elasticsearch Configuration

Elasticsearch was installed on the ELK server.

__The Elasticsearch service was verified using:__
```
sudo systemctl status elasticsearch
```
Elasticsearch was running with security enabled and HTTPS.

__The indexed logs were verified using:__
```
curl -k -u elastic "https://localhost:9200/nginx-logs-*/_count"
```
__The project successfully returned:__

`count: 145`

This confirmed that logs from the application servers were being indexed successfully.


## 11. Kibana Configuration

Kibana was installed on the ELK server and used as the monitoring interface.

__Kibana provides:__
* Log visualization
* Error monitoring
* Request analysis
* Client IP analysis
* Dashboard creation
* Alert configuration

__Kibana was accessed through:__
```
http://ELK-SERVER-IP:5601
```
The Elasticsearch index pattern was used to access the Nginx logs.

## 12. Kibana Visualizations

The following visualizations were created according to the project requirements.

### 1 Error Rate

This visualization shows HTTP error activity from the Nginx logs.

__It focuses on:__
```
4xx
5xx
```
# 📸 Screenshot  — Error Rate Visualization
![src7](./img/Screenshot%202026-10-03%20003253.png)
### 2. Requests per Minute

A visualization was created to analyze request activity over time.

This helps understand the number of requests being received by the application servers.

# 📸 Screenshot  — Requests per Minute

If your final dashboard does not show this panel, do not claim it is present. Keep the screenshot only if the visualization was actually completed.
![src6](./img/Screenshot%202026-10-03%20014136.png)
### 3. Top Client IPs

This visualization displays the client IP addresses generating requests.

It helps identify which clients are generating the highest number of requests.

# 📸 Screenshot 7— Top Client IPs
![src5](./img/Screenshot%202026-10-03%20020845.png)
## 13. Kibana Dashboard

The individual visualizations were combined into a centralized Kibana dashboard.

__The dashboard provides a single monitoring view for:__
```
HTTP error activity
Request activity
Top client IPs
```
The dashboard makes it easier to monitor application logs without checking each application server individually.

# 📸 Screenshot  — Final Kibana Dashboard
![src3](./img/Screenshot%202026-10-03%20124152.png)
![src4](./img/Screenshot%202026-10-03%20040021.png)
Take one screenshot showing the complete dashboard.

## 14. Alert Configuration

An alert was configured in Kibana to monitor HTTP errors.

__The alert query was:__
```
status >= 400 AND status < 600
```
__This includes:__
```
400–499 → 4xx errors
500–599 → 5xx errors
```
__The alert was configured to run every:__

`1 minute`

The configured threshold was:

`Above 5`

The alert query executed successfully.

__At the time of final verification, the alert showed:__
```
Alerts: 0
Active now: 0
```
This means the alert rule was configured and executing successfully, but there was no triggered alert at that verification time.

# 📸 Screenshot — Alert Configuration

Show:

Alert name: Nginx HTTP Error Alert
Query
Schedule: 1 minute
Threshold
Successful execution
![src2](./img/Screenshot%202026-10-03%20125130.png)
## 15. Complete Log Flow

__The final data flow of the project is:__
```
Nginx
   ↓
Access Logs
   ↓
Filebeat
   ↓
Logstash :5044
   ↓
Parse / Filter
   ↓
Elasticsearch :9200
   ↓
Kibana :5601
   ↓
Dashboard
   ↓
Alert
```
This provides centralized logging and monitoring for multiple application servers.

## 17. Verification

The complete pipeline was verified from the application servers to Kibana.

### Application Server Verification:

Nginx generated access logs successfully.

Filebeat was running and forwarding logs.

### Logstash Verification:

Logstash successfully validated the pipeline and processed incoming logs.

### Elasticsearch Verification:

The Elasticsearch index contained processed Nginx log documents.


### Kibana Verification:

Kibana successfully displayed the processed log data through visualizations and dashboard panels.

### Alert Verification:

The alert query executed successfully.

## 18. Project Deliverables

The following deliverables were completed according to the project requirements.

### 1. Log Pipeline Configuration

_Includes:_
```
Filebeat
Logstash
Elasticsearch
Nginx
```
### 2. Dashboard Screenshots

Kibana dashboard showing the available monitoring visualizations.
![src3](./img/Screenshot%202026-10-03%20124152.png)
![src4](./img/Screenshot%202026-10-03%20040021.png)

### 3. Alert Configuration Proof

`Kibana alert configuration for:`
```
status >= 400 AND status < 600
```
![src2](./img/Screenshot%202026-10-03%20125130.png)
## 19. Lessons Learned

__Through this project, the following concepts were practically implemented:__

1. Centralized logging using ELK.
2. Nginx log collection.
3. Filebeat log forwarding.
4. Logstash log processing.
5. Grok-based log parsing.
6. Elasticsearch indexing.
7. Kibana visualization.
8. HTTP 4xx and 5xx monitoring.
9. Kibana alert configuration.
10. Troubleshooting Elasticsearch and Logstash connectivity.
11. Managing ELK components on a limited-resource EC2 instance.

# Conclusion

This project demonstrates a centralized logging solution using the ELK Stack on AWS EC2.

Nginx logs from multiple application servers are collected using Filebeat and forwarded to Logstash. Logstash processes and parses the logs before storing them in Elasticsearch. Kibana provides centralized visualization and monitoring of the collected logs.

The system also includes an HTTP error alert mechanism for monitoring 4xx and 5xx responses.

This architecture reduces the need to manually inspect logs on individual servers and provides a centralized approach for application monitoring and troubleshooting.