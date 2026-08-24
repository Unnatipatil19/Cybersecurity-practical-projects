# 11. Wazuh Home Lab

## Quick Review of Tool
Wazuh is an open-source security monitoring platform that combines Endpoint Detection and Response (EDR) with Security Information and Event Management (SIEM) capabilities. It relies on lightweight agents deployed on target assets to collect system event telemetry, monitor file system integrity, detect rootkits, and scan for unpatched software vulnerabilities. 

The central manager analyzes this incoming telemetry, maps anomalous actions directly to the MITRE ATT&CK matrix, and executes automated active response rules to instantly contain active network threats.

---

## Server
The primary Wazuh central management dashboard instance was successfully built and deployed inside the network monitoring enclave using container management configurations.

```bash
\$ sudo docker-compose up -d
```
The central manager initialized its monitoring services, opening up the primary web interface endpoint across secure console port 443.

---

## Linux/Windows Agents
The host deployment agent software packages were installed on the asset targets across the local laboratory infrastructure.
* **Agent Service Verification**:
```bash
\$ sudo systemctl status wazuh-agent
● wazuh-agent.service - Wazuh Agent
     Loaded: loaded (/lib/systemd/system/wazuh-agent.service; enabled;)
     Active: active (running) since Thu 2026-08-20 10:14:02 UTC
```
The status logs confirm that the endpoint agent successfully verified communication parameters with the management server, enrolling the local node `192.168.207.138` into the active data pool.

---

## FIM
The File Integrity Monitoring (FIM) engine was enabled inside the core system layout configuration blocks to track critical operating directories in real-time.

```xml
<syscheck>
  <directories check_all="yes" realtime="yes">/etc,/usr/bin</directories>
</syscheck>
```
To audit detection fidelity, a testing alteration was made to the local configuration strings. The database flagged the modification instantly, generating an alert on the manager dashboard showing that `/etc/passwd` was modified.

---

## Vulnerability Detection
The endpoint collection agent regularly inventories installed system software packages and maps version strings against the National Vulnerability Database (NVD) index to check for system exposures.
* **Discovered Risks**: The agent tracking data flagged the target node asset running at `192.168.207.138` as highly vulnerable due to the presence of an outdated Apache framework component matching public exploit registry **CVE-2021-44228**.

---

## Rootcheck
The integrated Rootcheck security audit tool ran deep checks on the system to scan for hidden system anomalies, unauthorized kernel hooks, or persistent rootkit payloads.
* **Audit Results**: The system scanner completed its loop, flag-checking runtime processes and logging system permission tables to ensure a hardened baseline configuration.

---

## MITRE ATT&CK
The central logging engine automatically correlates incoming endpoint anomalies with known adversarial tactics listed in the industry-standard MITRE ATT&CK framework database.
* **Alert Instance**: High-frequency authentication failures hitting the secure network access service.
* **Framework Correlation**: Automatically categorized under **Credential Access tactics** and explicitly mapped to technique identifier **T1110 (Brute Force)**.

---

## Active Response
An automated active response policy block was configured on the central management engine to dynamically contain threat actors whenever specific rule definitions trigger high severity warnings.

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5712</rules_id>
  <timeout>600</timeout>
</active-response>
```
When an external host attempts to brute-force the system terminal parameters, the engine triggers this active response policy automatically, creating a temporary local firewall rule to completely drop incoming packets from that source IP address for 10 minutes.
