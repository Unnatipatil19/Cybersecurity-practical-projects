# 1. Network Scanning using Nmap

## Quick Review of Tool
Network scanning is the fundamental initial phase of any security assessment or penetration test. It allows an engineer to map out an active network environment by sending specialized packets to target IP addresses and analyzing the responses. This process identifies active devices, maps open communication pathways, and gathers intelligence on potential target exposures.

Nmap (Network Mapper) is the industry-standard, open-source utility used for network discovery and security auditing. It utilizes raw IP packets in creative ways to determine what hosts are available on the network, what services (including application name and version details) those hosts are offering, what operating systems they are running, what type of packet filters or firewalls are in use, and whether any known vulnerabilities exist.

---

## Step 1: Host Discovery
Before running comprehensive port scans, a basic ICMP echo request probe was executed to confirm that the target system is live, responsive, and available on the local subnet path.

```bash
kali@kali:~/Downloads\$ ping -c 4 192.168.207.138
PING 192.168.207.138 (192.168.207.138) 56(84) bytes of data.
64 bytes from 192.168.207.138: icmp_seq=1 ttl=64 time=1.60 ms
64 bytes from 192.168.207.138: icmp_seq=2 ttl=64 time=0.561 ms
64 bytes from 192.168.207.138: icmp_seq=3 ttl=64 time=0.432 ms
64 bytes from 192.168.207.138: icmp_seq=4 ttl=64 time=0.368 ms

--- 192.168.207.138 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3060ms
rtt min/avg/max/mdev = 0.368/0.739/1.595/0.499 ms
```

The target responded successfully with zero packet loss, confirming the host is online and ready for active scanning.

---

## Step 2: Port Scanning
A stealthy TCP SYN port scan (`-sS`) was launched against the target IP to probe the top 1000 standard ports and discover which communication channels are open.

```bash
kali@kali:~/Downloads\$ sudo nmap -sS 192.168.207.138
[sudo] password for kali: 

Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-27 10:12 EDT
Nmap scan report for 192.168.207.138
Host is up (0.0019s latency).
Not shown: 997 closed tcp ports (reset)

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https

Nmap done: 1 IP address (1 host up) scanned in 1.45 seconds
```

The scan reveals three open entry ports: 22 (SSH), 80 (HTTP), and 443 (HTTPS).

---

## Step 3: Service & Version Detection
With the open ports identified, a service detection scan (`-sV`) was performed to determine the exact software applications and specific versions running behind those open channels.

```bash
kali@kali:~/Downloads\$ sudo nmap -sV 192.168.207.138

Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-27 10:13 EDT
Nmap scan report for 192.168.207.138
Host is up (0.0020s latency).
Not shown: 997 closed tcp ports (reset)

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
443/tcp  open  ssl/http Apache httpd 2.4.52 ((Ubuntu))

Nmap done: 1 IP address (1 host up) scanned in 11.20 seconds
```

---

## Step 4: OS Detection
An operating system identification scan (`-O`) was performed to analyze the TCP/IP stack fingerprinting responses and determine the remote OS running on the virtual instance.

```bash
kali@kali:~/Downloads\$ sudo nmap -O 192.168.207.138

Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-27 10:14 EDT
Nmap scan report for 192.168.207.138
Host is up (0.0021s latency).
Not shown: 997 closed tcp ports (reset)

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
MAC Address: 00:0C:29:5F:A4:B2 (VMware Virtual Machine)

Device type: general purpose
Running: Linux 5.X
OS splinter-guesses: Linux 5.4 - 5.11 (96%)
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org .
Nmap done: 1 IP address (1 host up) scanned in 5.82 seconds
```

---

## Step 5: NSE Script Scanning
The Nmap Scripting Engine was invoked using the `--script=vuln` parameter to automatically query the discovered application services against a known database of configuration flaws and software bugs.

```bash
kali@kali:~/Downloads\$ sudo nmap -sV --script=vuln 192.168.207.138

Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-27 10:15 EDT
Nmap scan report for 192.168.207.138
Host is up (0.0021s latency).
Not shown: 997 closed tcp ports (reset)

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-dombased-xss: Vulnerable to DOM-Based XSS
443/tcp  open  ssl/http Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
| ssl-dhparam: 
|   VULNERABLE:
|   Diffie-Hellman Key Exchange Insufficient Key Strength
|     State: VULNERABLE
|     Risk factor: High
|     Description:
|       The server uses a Diffie-Hellman group with an insufficient key size (1024 bits).

Nmap done: 1 IP address (1 host up) scanned in 15.34 seconds
```

---

## Step 6: Firewall Detection
A TCP ACK probe scan (`-sA`) was executed against the target system to determine if an active local packet filter or firewall rule is protecting the pathways.

```bash
kali@kali:~/Downloads\$ sudo nmap -sA 192.168.207.138

Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-27 10:16 EDT
Nmap scan report for 192.168.207.138
Host is up (0.0018s latency).
All 1000 scanned ports on 192.168.207.138 are Class: unfiltered (reset)

Nmap done: 1 IP address (1 host up) scanned in 4.12 seconds
```

The scan returned "unfiltered" for all tracked channels, meaning there is no active host-based firewall policy blocking connection vectors on this network segment.

---

## Scan Report & Recommend Mitigations
* **Target Node Assessment**: Host `192.168.207.138` is verified online inside the VMware virtual lab network.
* **Exposed Environment**: Discovered three open system interfaces including an SSH administration terminal (Port 22) and Apache web platform infrastructure (Ports 80 and 443).
* **Identified Risks**: The scan detected clear banner leaks disclosing software versions, a critical DOM-based Cross-Site Scripting (XSS) vulnerability on the port 80 application layer, and an insecure 1024-bit Diffie-Hellman cryptographic parameter on port 443.

### Recommended Defensive Hardening Strategy
1. **Remediate Cryptographic Key Exchange Vulnerability**: Modify the web application configuration files on port 443 to reject Diffie-Hellman prime group parameters under 2048 bits or migrate to elliptic curve cipher structures.
2. **Mitigate Web Application Injection Vectors**: Review the site scripts operating on port 80 to implement strict context-aware output encoding and sanitize all user input vectors to neutralize DOM-based XSS exploits.
3. **Restrict Application Banner Leakage**: Modify the `/etc/apache2/apache2.conf` security policy file to set `ServerTokens ProductOnly` and `ServerSignature Off`. This hides software version data from scanning utilities.
  
