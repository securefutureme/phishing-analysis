# Email 01 - Mass-phishing using urgency tactics

Even at a quick glance — from an average recipient’s perspective — two elements jump out immediately: the alarmist subject line “There’s no time left” and the claimed sender “UPS”.
This combination alone triggers an instinctive brake: an extreme sense of urgency usually doesn’t fit the typical, calm communication style of courier companies.

- **UPS does not use messages like “there’s no time left”; rather “Your parcel is ready for pickup”, “Your parcel is on its way”, etc.**
- **I wasn’t using UPS services at the time I received the message, which is an additional red flag.**

<p align="center">
  <a href="screenshots/mail1/header1.png">
    <img src="/screenshots/mail1/header2.png" width="600" alt="Widok maila (nagłówek)">
  </a>
</p>
<p align="center">
  <em>. 1 — Headers</em>
</p>

Digging deeper—viewing the message preview in the sandbox—the email immediately looks like classic phishing, **mainly because it is very poorly crafted.**
The content contains no elements confirming it is actually UPS (no consistent sender domain, no shipment identifiers).
The first **FROM** header confirms a classic case of display-name spoofing and impersonation of a well-known company **(brand impersonation)**.

A quick inspection of the content and embedded links (after opening the message as .txt) indicates an attempt at fraud: **the visible “U.n.s.u.b” link points directly to the IP address 5[.]231[.]202[.]248,** and the code includes a tracking pixel from the domain chipcrack[.]net—neither has any connection to the UPS domain. The domain/brand mismatch and the use of a raw IP address in a CTA link are classic red flags of bulk phishing campaigns.
<div align="center">
  <img src="/screenshots/mail1/message1.png" alt="Podgląd wiadomości" width="35%">
  &nbsp;&nbsp;
  <img src="/screenshots/mail1/screenbody1.png" alt="Widok HTML" width="55%">
</div>
<p align="center"><em>Fig. 2,3 — Message view + HTML</em></p>

Upon further analysis of the email, we can see this is not UPS correspondence but a bulk campaign sent from the attacker’s own infrastructure. The first red flag—besides the spoofed display name—is **Return-Path: <…@chipcrack.es>**: the bounce address also points to chipcrack.es. The extremely long local-part (a string of digits/letters) looks like a tracking tag/campaign identifier (VERP/track ID).

Next, we look at the header **Received: from aaa[.]altnewlywed[.]shop ([163[.]172[.]189[.]190])**. This shows the message entered from host `aaa[.]altnewlywed[.]shop` at public IP `163[.]172[.]189[.]190`. That appears to be a standard VPS with a random domain—definitely not UPS infrastructure.

I then verified DKIM and SPF. The signatures are issued for `chipcrack[.]es` (not `ups.com`, which is further evidence of phishing) and they use SHA-1 (a weak and outdated algorithm). A DKIM signature for the attacker’s domain does not prove the supposed brand’s authenticity—only that the sender controls `chipcrack[.]es`. **The email also lacks SPF/DKIM/DMARC results from the receiving server—i.e., there is no `Authentication-Results` section.** This can still be normal behavior: AV/antispam gateways sometimes strip or replace `Authentication-Results:` with their own `X-*` headers. Mailing lists may also leave only their metadata. It’s also possible that older mail systems don’t run SPF/DKIM/DMARC modules at all or don’t log results in `Authentication-Results`.

Another notable point: the email embeds an image link hosted on `imgur.com`. After opening the link, the image is not present on the server, which further undermines credibility. A legitimate sender (e.g., UPS) **would not host email assets on a public service like imgur.com—they would use their own domain.** The missing file suggests a “one-off” campaign (rotation/cleanup of assets, removal by Imgur or the author), typical of bulk phishing: a sloppily assembled template, external hosts, and short-lived artifacts. Additional checks in urlscan.io, WHOIS, and MXtoolbox confirm this was a bulk phishing run carried out some time ago.

## Summary

- **Objective:** Redirect to a campaign site (likely data/payment theft—cannot be fully verified due to the campaign being inactive).
- **Attack type:** Likely bulk phishing with brand spoofing and display-name spoofing.
- **Final assessment:** _**Phishing**_

### URL analysis

##### urlscan.io
**hxxp://chipcrack[.]net/track/.../30285o9**
- This is an **open-tracking pixel** embedded in the phish (path `/track/...` + 1×1 PNG). It’s used to confirm whether a mailbox is “alive” and the email was viewed. It also tracks the campaign (ID in the path) and can be used for follow-up spamming/targeting.

**5[.]231[.]202[.]248**
- The scan returns an error (“We could not scan this website!”). The link likely no longer works because the campaign was retired or the IP/server stopped responding / is blocking scanners.

##### VirusTotal 
<div align="center">
  <img src="/screenshots/mail1/virustotal.png" alt="Widok HTML" width="105%">
</div>

##### WHOIS 
**5[.]231[.]202[.]248**
- **RIPE region (RIPE NCC).** Owner details can be retrieved from the RIPE (RDAP) database. The address does not belong to UPS. It’s likely an intermediate/VPS server used in the campaign.

##### MXToolbox 
- According to the report, the domain has **poor email hygiene** and does not look like an official UPS sending channel — **no DMARC**, missing/invalid **SPF**, and issues with **MX/DNS** records.
<div align="center">
  <img src="/screenshots/mail1/mxtoolbox.png" alt="Widok HTML" width="105%">
</div>


### Header Analysis

| Field                     | Value                                                                              | Note |
|---                        |---                                                                                 |---|
| `From`                    | `"UPS"` <…@chipcrack[.]es>                                                         | **Display-name spoofing** |
| `Reply-To`                | —                                                                                  | n/a |
| `Return-Path`             | <…@chipcrack[.]es>                                                                 | Bounce address points to the campaign sender’s domain. |
| `Received` (last hop)     | from aaa.altnewlywed[.]shop ([163.172.189.190])                                    | Last host is a VPS with an unrelated domain; no link to UPS. |
| **SPF**                   | n/a                                                                                | n/a |
| **DKIM**                  | `d=chipcrack[.]es; a=rsa-sha1; s=smtp`                                             | Signature for the attacker’s domain, not UPS. |
| **DMARC**                 | n/a                                                                                | n/a |

### IOC Table

| **Type**   | **Value**                                                                                              | **Context**                               | **Confidence** |
|---         |---                                                                                                     |---                                         |---|
| **Domain** | chipcrack[.]es                                                                                        | Envelope-From / Return-Path / DKIM `d=`    | High |
| **Domain** | aaa[.]altnewlywed[.]shop                                                                              | Sending host in `Received`                 | High |
| **IP**     | 163[.]172[.]189[.]190                                                                                 | Source IP of the sending server            | High |
| **URL**    | hxxp://5[.]231[.]202[.]248/ShpVWA32333Tpuf133gwzwgzcmey1491.../3028529                                | CTA / “U.n.s.u.b” from the email           | High |
| **Domain** | chipcrack[.]net                                                                                       | Campaign tracker/pixel                     | Medium |
| **URL**    | hxxp://chipcrack[.]net/track/3zpkFh32333PFCr133uvidiswzhm1491.../30285o9                              | Campaign tracker/pixel                     | Medium |
| **Subject**| “nie ma już czasu” ("there’s no time left")                                                               | Time-pressure tactic (social engineering)  | Medium |
| **Phrase** | “If you no longer wish to receive emails from us, please U.n.s.u.b”                                   | Campaign phrase                            | Medium |
