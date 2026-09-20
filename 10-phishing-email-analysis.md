# 10. Phishing Email Analysis

## Objective
To analyze an email's headers and check whether it is legitimate or suspicious, using the SPF, DKIM, and DMARC authentication results.

## Tools Used
- Gmail (Original message view)
- MXToolbox Email Header Analyzer (mxtoolbox.com/EmailHeaders.aspx)

## Email Analyzed
- **Subject:** Slide Maker just landed — and so did AI Drive
- **Sender domain:** gpai.app
- **Found in:** Gmail Spam folder

## Procedure
1. Opened the email from the Spam folder in Gmail without clicking any links.

   ![Email in Gmail](<img width="997" height="577" alt="1st email" src="https://github.com/user-attachments/assets/aa30d37a-d359-4da5-989d-8f759ee99451" />

2. Opened the **Original message** view to see the full headers and Gmail's authentication summary.

   ![Original message]<img width="1133" height="418" alt="2nd original messsage" src="https://github.com/user-attachments/assets/9f330e6b-d2c3-4349-b352-4c13cbbd0b34" />

3. Copied the headers and pasted them into the MXToolbox Email Header Analyzer.

   ![MXToolbox input]<img width="1319" height="524" alt="mxtool result" src="https://github.com/user-attachments/assets/0cc5ce0b-84e7-412c-be40-e27c92ccf5be" />
4. Clicked **Analyze Header Results** and reviewed the Delivery Information section.

   ![MXToolbox results<img width="1357" height="640" alt="result" src="https://github.com/user-attachments/assets/4067a6b0-5b3b-4d12-8560-267cfc61a1b5" />

## Results

### Gmail (Original message)
| Check | Result |
|-------|--------|
| SPF | PASS |
| DKIM | PASS |
| DMARC | PASS |

### MXToolbox (Delivery Information)
| Check | Result |
|-------|--------|
| SPF Alignment | Pass |
| SPF Authenticated | Pass |
| DKIM Alignment | Pass |
| DKIM Authenticated | Pass |
| DMARC Compliant | Fail (policy not enforced) |

## Findings
- The email passed SPF, which means it was sent from a server authorized by the sender's domain.
- It passed DKIM, which means the message was not changed in transit.
- Gmail reported DMARC as PASS, but MXToolbox flagged the domain's DMARC policy as not enforced. This means the domain monitors spoofing but does not tell mail servers to reject fake emails.
- Based on these checks, the email is most likely a legitimate marketing email and not phishing.
- The weak DMARC policy is still a risk, because attackers could more easily spoof this domain.

## Conclusion
Checking SPF, DKIM, and DMARC helps separate legitimate emails from suspicious ones. No single check is enough. The sender address, links, and email content should also be reviewed.
