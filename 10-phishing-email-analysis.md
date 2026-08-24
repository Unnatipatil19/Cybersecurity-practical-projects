# 10. Phishing Email Analysis

## Quick Review of Tool
Email infrastructure remains a primary access vector for modern threat campaigns. Phishing email analysis involves evaluating incoming messages to verify authenticity, isolate credential-harvesting mechanisms, and extract Indicators of Compromise (IOCs). 

Analysts dissect raw email records to inspect transmission header pathways and check security authentication frameworks like Sender Policy Framework (SPF), DomainKeys Identified Mail (DKIM), and Domain-based Message Authentication, Reporting, and Conformance (DMARC). This process traces deceptive links and identifies macro-enabled malicious attachments to protect organizational data systems.

---

## Headers
A suspicious file sample was loaded into an authentication tracking tool to inspect routing records inside the message header blocks.

```text
Received: from mail.attacker-domain.xyz (unverified)
          by mx.lab-mail-gateway.local with ESMTPS;
          11:14:22
From: "Security Alert Bank" <support@secure-bank-update.com>
To: victim@lab-organization.com
Subject: Immediate Action Required: Account Suspended!
```
The header evaluation shows that while the visible sender name claims to represent an official financial institution, the actual sending envelope domain belongs to an unrelated external source.

---

## SPF/DKIM/DMARC
The domain protection records were audited within the authentication results field to check message origin validation logs.

```text
Authentication-Results: mx.lab-mail-gateway.local;
       spf=fail (domain of support@secure-bank-update.com does not designate 203.0.113.50 as permitted sender)
       dkim=fail (signature verification failed)
       dmarc=fail (p=REJECT)
```
The message failed all three verification boundaries, confirming that the sender identity was spoofed and did not originate from an authorized mail gateway.

---

## URLs
The hyperlinks embedded within the plaintext and HTML body frames were parsed to check their actual target destinations.
* **Visible Text in Body**: `https://officialbank.com`
* **Actual Underlying Destination URL**: `http://secure-bank-update.com`

The analysis reveals an intent to deceive; clicking the link routes unsuspecting users straight to an external credential-harvesting page.

---

## Attachments
A file attachment named `Invoice_Document.pdf.exe` was extracted within a safe laboratory sandbox to perform initial static file verification checks.
* **Analysis Observation**: The sender utilized a dangerous double-extension naming trick to make an active Windows executable binary disguise itself as a benign PDF document file.
* **Cryptographic Hash Generation**:
```bash
$ sha256sum Invoice_Document.pdf.exe
a1b2c3d4e5f60123456789abcdef99887766554433221100fedcba9876543210
```

---

## IOCs
The primary Indicators of Compromise (IOCs) isolated from the malicious email artifact are logged below for defensive filtering updates:
* **Sending IP Gateway Node**: `203.0.113.50`
* **Deceptive External Domain**: `secure-bank-update.com`
* **Malicious URL string**: `http://secure-bank-update.com`
* **Attachment Signature (SHA256)**: `a1b2c3d4e5f60123456789abcdef99887766554433221100fedcba9876543210`

---

## Report
The complete investigation findings were structured into a standard incident response assessment report for enterprise defense teams.

```text
INCIDENT ASSESSMENT REPORT - PHISHING ESCALATION
-----------------------------------------------------------
Threat Vector Profile : Inbound Deceptive Phishing Campaign
Authentication Metric : SPF/DKIM/DMARC Core Alignment Failures
Impact Level Summary  : High Risk - Credential Harvesting and Malware Delivery
Recommended Action    : Block sender IP 203.0.113.50 and domain secure-bank-update.com at the email gateway. Remove matching messages from all mailboxes, and add the attachment file hash to endpoint security blocklists.
```
