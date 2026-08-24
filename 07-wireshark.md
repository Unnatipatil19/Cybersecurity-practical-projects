# 7. Wireshark

## Quick Review of Tool
Wireshark is an industry-standard network protocol analyzer used to capture and interactively browse the traffic running on a computer network. It sniffs raw packets moving across a network interface card in real time and decodes their structure into human-readable data frames, tracking protocols across all layers of the OSI model. 

By applying specialized display filters, a security analyst can trace individual communication paths, evaluate application-layer headers, verify encryption implementation, and isolate malicious data payloads or network flood anomalies during an incident investigation.

---

## Packet Capture
I opened the Wireshark application interface on my local terminal, highlighted the primary active virtual network adapter card interface, and initiated a live network packet capture session to monitor ongoing communications.

---

## TCP Handshake
I generated a network connection attempt to audit transmission establishment parameters and applied a packet filter to isolate the precise three-way initialization handshake sequence.
* **Packet 1 (SYN)**: Source `192.168.207.129` -> Destination `192.168.207.138` `[SYN] Seq=0 Win=64240 Len=0`
* **Packet 2 (SYN-ACK)**: Source `192.168.207.138` -> Destination `192.168.207.129` `[SYN, ACK] Seq=0 Ack=1 Win=65160 Len=0`
* **Packet 3 (ACK)**: Source `192.168.207.129` -> Destination `192.168.207.138` `[ACK] Seq=1 Ack=1 Win=64240 Len=0`

The sequence confirms a successful, reliable connection sync between the local testing nodes.

---

## DNS
I modified the active session filters to track local Domain Name System address lookup operations routing traffic across network names.
```text
Source              Destination         Protocol  Info
192.168.207.138     192.168.207.2       DNS       Standard query 0x1a42 A updates.kali.org
192.168.207.2       192.168.207.138     DNS       Standard query response 0x1a42 A updates.kali.org 192.124.249.10
```

---

## HTTP/HTTPS
I analyzed the transport layer security configurations by generating and comparing plaintext web requests against secure, encrypted connections.
* **HTTP Assessment**: Full payload data visibility. Request fields, method parameters, and session strings were exposed in cleartext.
* **HTTPS Assessment**: Complete data protection. The packet payload layers returned unreadable random hexadecimal bytes, showing that the communication channel was wrapped securely inside an encrypted Transport Layer Security container.

---

## ICMP
I monitored local diagnostic ping utilities to verify how the network adapters process connectivity checks. The application logs correctly paired each independent inbound request data line directly with its corresponding response packet return, measuring the exact transmission latency.

---

## FTP
I ran a test file transfer session across a legacy, unencrypted File Transfer Protocol service to document protocol credential vulnerability data.
```text
220 Welcome to Laboratory Storage Node
USER administrator
331 Password required for administrator.
PASS P@ssword_Testing_Secure
230 User logged in, proceed.
```
Because the underlying protocol lacks encryption controls, all authentication strings were fully intercepted in cleartext inside the Wireshark packet view layers.

---

## Attack Analysis
An archived network capture trace file was imported into the analyzer to isolate a Denial of Service attack pattern targeting the web infrastructure.
* **Identified Anomaly**: The packet timeline data recorded an aggressive, high-density flood of repeating `[SYN]` connection requests targeting port 80 on node `192.168.207.138` within a fraction of a second.
* **Diagnostic Summary**: The pattern confirms a textbook **TCP SYN Flood Attack**, designed to exhaust system socket tables and force a service disruption condition on the web target.
