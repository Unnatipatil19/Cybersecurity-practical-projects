# 9. Splunk Log Analysis

## Quick Review of Tool
Security Information and Event Management (SIEM) platforms are critical for maintaining visibility across enterprise networks. Splunk is an industry-leading log aggregation and analytics engine that centralizes machine data from diverse infrastructure endpoints, including web servers, operating systems, and network firewalls. 

By utilizing Search Processing Language (SPL), analysts can write focused queries to parse millions of raw log entries in real-time. This allows defensive teams to detect authentication anomalies, map brute-force indicators, correlate events, and deploy security dashboards for security operational monitoring.

---

## Install
The Splunk Enterprise core engine was extracted and initialized inside the management workspace using the command-line interface.

```bash
$ sudo ./splunk start --accept-license
```
The application successfully spawned the background monitoring daemons and opened the secure operational web interface on standard administration port 8000.

---

## Add Data
The ingestion manager was configured to parse system data paths from the target asset.
* **Target Log Sources**: `/var/log/auth.log` (System Authentication) and `/var/log/apache2/access.log` (Apache Web Traffic).
* **Source Type Configuration**: Automated event parsing with standard line-breaking structures.

---

## SPL
I used Search Processing Language (SPL) statements within the main repository index to isolate system telemetry associated with the lab asset IP address.

```text
index=main source="/var/log/auth.log" 192.168.207.138
```
The query filtered thousands of baseline logs down to the specific transactional rows belonging to the target network node.

---

## Failed Logins
I structured an SPL search statement targeting the authentication data source to extract failed login actions and pinpoint potential unauthorized access trails.

```text
index=main source="/var/log/auth.log" "failed" OR "Failure"
| table _time, user, src_ip, message
```

### Generated Event Records
```text
_time                user     src_ip           message
-----------------------------------------------------------------------------------------
14:22:02             root     192.168.207.129  Failed password for invalid user root
14:22:05             root     192.168.207.129  Failed password for invalid user root
```

---

## Brute Force
To filter out accidental password typos from active automated scanning scripts, I expanded the SPL search parameters to flag high-frequency login exceptions coming from a single source node.

```text
index=main source="/var/log/auth.log" "Failed password"
| stats count as failure_count by src_ip, user
| where failure_count > 15
```
This data calculation aggregates total event instances and flags any external IP address triggering more than 15 consecutive authentication failures against system user accounts.

---

## Dashboards
The validated security queries were transformed into visual objects on a centralized operations dashboard:
* **Panel A**: A real-time data chart tracking the total frequency of authentication failures to visualize sudden attack spikes.
* **Panel B**: A metrics table highlighting the top targeted usernames to flag credential spraying patterns.

---

## Alerts
The brute-force query logic was saved as an automated metric tracking rule inside the Splunk alerts engine.
* **Trigger Threshold**: Trigger condition met if total occurrences exceed 15 hits within a sliding 5-minute tracking window.
* **Configured Automated Action**: Write the event to the primary security console and trigger an automated administrative log notification.
