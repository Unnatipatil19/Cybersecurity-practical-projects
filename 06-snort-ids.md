# 6. Snort IDS

## Quick Review of Tool
Snort is an open-source Network Intrusion Detection System (NIDS) used to monitor network traffic in real time. It analyzes packets passing through a network interface card and matches them against a defined list of custom rules. If any traffic matches a rule signature, Snort immediately throws a security alert to flag the suspicious activity.

---

## Install
The Snort utility framework package was deployed onto the local network interface card using standard administration package tools.

```bash
$ sudo apt-get update && sudo apt-get install snort -y
```

---

## Configure
The primary configuration settings inside the core rules file were edited to define the home network address parameters and link out to local tracking files.

```text
$ sudo cat /etc/snort/snort.conf

ipvar HOME_NET 192.168.207.0/24
ipvar EXTERNAL_NET any
var RULE_PATH /etc/snort/rules
include $RULE_PATH/local.rules
```

---

## Custom Rules
Three distinct custom tracking signatures were appended to the rules engine file to flag explicit threat profiles targeting the laboratory network asset.

```text
$ sudo cat /etc/snort/rules/local.rules

alert icmp any any -> 192.168.207.138 any (msg:"ICMP Ping Probe Detected"; sid:1000001; rev:1;)
alert tcp any any -> 192.168.207.138 any (msg:"Nmap TCP Stealth Scan Flag Pattern"; flags:S; sid:1000002; rev:1;)
alert tcp any any -> 192.168.207.138 22 (msg:"SSH Brute Force Attack Pattern"; flags:S; threshold:type threshold, track by_src, count 5, seconds 30; sid:1000003; rev:1;)
```

---

## ICMP
The Snort inspection engine daemon was initialized in the active terminal space to print incoming security events to the screen instantly upon validation.

```bash
$ sudo snort -A console -q -c /etc/snort/snort.conf -i eth0
08/08-14:20:11.450210 [**] [1:1000001:1] ICMP Ping Probe Detected [**] {ICMP} 192.168.207.1 -> 192.168.207.138
```

---

## Port Scan
The console captured live TCP flags matching structural stealthy port scan tracking triggers launched from an external network testing node.

```text
08/08-14:22:35.109432 [**] [1:1000002:1] Nmap TCP Stealth Scan Flag Pattern [**] {TCP} 192.168.207.129:38420 -> 192.168.207.138:80
08/08-14:22:35.109855 [**] [1:1000002:1] Nmap TCP Stealth Scan Flag Pattern [**] {TCP} 192.168.207.129:38422 -> 192.168.207.138:443
```

---

## SSH Brute Force
The threshold monitoring limits successfully identified a high-frequency connection burst aimed directly at the secure terminal access portal on port 22.

```text
08/08-14:25:01.890432 [**] [1:1000003:1] SSH Brute Force Attack Pattern [**] {TCP} 192.168.207.129:41202 -> 192.168.207.138:22
```

---

## Alerts
The background execution task was stopped, and the standard storage logging folders were audited to verify that all validated anomaly records were written correctly.

```bash
\$ sudo tail -n 2 /var/log/snort/alert
[**] [1:1000003:1] SSH Brute Force Attack Pattern [**] [Priority: 0] 08/08-14:25:01 192.168.207.129 -> 192.168.207.138
[**] [1:1000001:1] ICMP Ping Probe Detected [**] [Priority: 0] 08/08-14:26:15 192.168.207.1 -> 192.168.207.138
`
