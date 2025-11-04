# Email 02 - Credential Harvesting

**Email used from malware-traffic-analysis.net ------- For training / educational purposes only**

Starting the analysis of an email pulled from an excellent site that hosts collections of malware and phishing emails (which is an amazing resource for anyone looking to practice the analytical skills needed for SOC and beyond), we can see right away that the message was crafted to grab our attention. If we put ourselves in the shoes of a regular user who opens their inbox and receives such an email (assuming we’re familiar with malware-traffic-analysis.net), we’re told that the message came from the site’s Support team.

The first red flag here is the mismatch between the sender display name “malware-traffic-analysis.net Support” and the actual sender address sues@nnwifi[.]com. From a user’s perspective, “Support” sounds trustworthy, but the mail client shows the truth: the sender is sues@nnwifi[.]com, which is a completely different domain than the one used in the sender’s display name.

Add to this the “Warning: Final notice” — a classic social-engineering tactic to create time pressure. We can already see two major red flags to note in a phishing analysis.

<p align="center">
  <img src="/screenshots/mail2/header1.png" alt="Podgląd nagłówka" width="60%">
</p>
<p align="center">
  <em>Rys. 1 — Nagłówki</em>
</p>

Digging further into the email, the message body asks to “**confirm address ownership**” via a link that leads outside the recipient’s domain. Hovering over the link reveals the **destination**: `hxxps://servervirto[.]com[.]co/ed/trn/update?email=brad@malware-traffic-analysis.net`. The Email field is likely prefilled from the `email=` parameter (tracking), so the link probably redirects to a fake login page with the address already filled in to harvest passwords.

The address `admin@malware-traffic-analysis.net` in the footer is just plain text (no `mailto:`) — only styled in blue.

<p align="center">
  <img src="/screenshots/mail2/body1.png" alt="Podgląd nagłówka" width="60%">
  </p>
  <p align="center">
    <em>Rys. 2 — Message view</em>
</p>

However, this is one of the older emails from malware-traffic-analysis.net; the link no longer works today and returns a **400 error**. We can assume (given these are samples that may have been modified by malware-traffic-analysis) that either the links are not “live,” or the campaign has been retired, or the hosts are blocking scanners.  
Still, on malware-traffic-analysis.net we see a screenshot showing what appears after clicking “Confirm ownership”:

<p align="center">
  <img src="/screenshots/mail2/phishing-page.png" alt="Podgląd nagłówka" width="60%">
  </p>
  <p align="center">
    <em>Rys. 3 — Fake login page</em>
</p>

Ewidentnie mamy tutaj przykład **wyłudzania poświadczeń (credential phishing).**

This is clearly an example of **credential theft (credential phishing).**

From the message headers we gather more evidence. The last hop is `mail[.]nnwifi[.]com (173[.]46[.]174[.]49)`—that’s the server that delivered the message to the recipient. Below it we see several localhost (127.0.0.1) hops via amavisd-new (a kind of “go-between” on the same host, sitting between the MTA, e.g., Postfix, and content scanners); these are not separate Internet servers, just internal passes within `mail.nnwifi.com`. The most important is the lowest entry:

- **Received: from nnwifi[.]com (94[.]100[.]31[.]27) by mail[.]nnwifi[.]com with ESMTP**

Put simply: the message was submitted from host `nnwifi[.]com` (IP `94[.]100[.]31[.]27`) using ESMTP (i.e., authenticated client submission). **So someone logged in to `mail[.]nnwifi[.]com` and sent the email as “Support,” after which the local MTA ran it through its filters and relayed it onward.** This looks like a standard sender server for the campaign—**not** malware-traffic-analysis[.]net infrastructure. Of course, 100% certainty would require server logs, but based on the supplied headers, the theory of an external “legitimate” provider or a simple forward is **unlikely**.

**Three different domains in a single message let us confirm this is phishing.** The sender (`nnwifi.com`), the brand in the display name (`malware-traffic-analysis[.]net`—noting this is a sample/possibly modified email), and the link target (`servervirto[.]com[.]co`). Legitimate notifications rarely mix that many namespaces.

---

## Summary
- **Objective:** Impersonate mailbox support to steal credentials via a fake login page.
- **Attack type:** **Credential phishing** with **brand impersonation** and **display-name spoofing**.
- **Final assessment:** _**Phishing**_

### URL Analysis

##### urlscan.io
**servervirto[.]com**
- The link is currently non-functional—as is the entire `servervirto` domain.

##### WHOIS 
**173[.]46[.]174[.]49**
Last hop named `mail.nnwifi.com`—looks like a standard hosting provider’s MTA (geolocation: USA). This is not infrastructure of the impersonated domain.

##### MXToolbox 
**No DKIM/DMARC/SPF**

### Header Analysis

| Field                     | Value (sanitized)                                                                                               | Note |
|---                        |---                                                                                                              |---|
| `From`                    | `"malware-traffic-analysis.net Support"` <sues@nnwifi[.]com>                                                   | **Display-name spoofing** |
| `Return-Path`            | n/a (not provided)                                                                                              | No explicit bounce channel in the excerpt. |
| `Reply-To`               | n/a (not provided)                                                                                              | Often differs in phishing—no data here. |
| `Received` (last hop)    | from **mail.nnwifi[.]com** ([173[.]46[.]174[.]49])                                                              | Final sender server for the campaign. |
| **SPF**                  | n/a                                                                                                              | No `Authentication-Results` present in the provided fragment. |
| **DKIM**                 | n/a                                                                                                              | Same as above. |
| **DMARC**                | n/a                                                                                                              | Same as above. |


### IOC Table

| Type    | Value                                                                                                   | Context                                        | Confidence |
|---      |---                                                                                                      |---                                             |---|
| Domain  | nnwifi[.]com                                                                                            | Sender domain in `From` / `mail.nnwifi` hosts  | High       |
| Domain  | mail.nnwifi[.]com                                                                                       | Last hop in `Received`                         | High       |
| IP      | 173[.]46[.]174[.]49                                                                                     | IP of `mail.nnwifi[.]com`                      | High       |
| IP      | 94[.]100[.]31[.]27                                                                                      | IP of `nnwifi[.]com` (ESMTP submission)        | High       |
| URL     | hxxps://servervirto[.]com[.]co/ed/trn/update?email=brad@malware-traffic-analysis[.]net                  | “Confirm ownership” CTA link                   | High       |
| Domain  | servervirto[.]com[.]co                                                                                  | Campaign target (credential phishing)          | High       |
| Subject | `Warning: Final notice`                                                                                 | Time-pressure social engineering               | Medium     |
| Phrase  | `Confirm ownership`                                                                                     | Lure phrase (account verification)             | Medium     |
| Email   | sues@nnwifi[.]com                                                                                       | Sender address inconsistent with brand         | High       |
