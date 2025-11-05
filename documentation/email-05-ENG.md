# Email 05 — Deceptive phishing

### Source: [www.monkey.org](https://monkey.org/~jose/phishing/)

I came across this sample while browsing one of the most famous public phishing collections — the Nazario phishing corpus. It’s excellent training material for SOC teams — hundreds of real campaigns that let you practice analytical skills without the risk of touching a live victim infrastructure. As usual, I opened the message in an isolated environment. At first glance, it invokes the well-known “subscription renewal” theme; however, the email itself is fairly well executed:

<div align="center">
  <img src="/screenshots/mail5/message1.png" alt="Widok HTML" width="45%">
</div>
<p align="center"><em>Fig. 1 — Mail View</em></p>

Here, we’re dealing with phishing impersonating the **Netflix** brand. Why classify it as phishing right away? Because the first red flag is obvious:

<div align="center">
  <img src="/screenshots/mail5/header1.png" alt="HTML view" width="55%">
</div>
<p align="center"><em>Fig. 2 — Headers</em></p>

The message content doesn’t match the recipient address. Netflix does not send renewal emails from addresses outside its own domain. Here’s an example from my private mailbox:

<div align="center">
  <img src="/screenshots/mail5/orginalnetfl.png" alt="HTML view" width="55%">
</div>
<p align="center"><em>Fig. 3 — Header of a legitimate Netflix email</em></p>

A final user-visible proof is the CTA link — **“Update now”** — which clearly does **not** lead to any Netflix domain:
<div align="center">
  <img src="/screenshots/mail5/message3.png" alt="HTML view" width="55%">
</div>
<p align="center"><em>Fig. 4 — CTA link target</em></p>

The link currently returns no content (server responds **404**): `hxxps://pos[.]nham24[.]com/files/files/help[.]html`. The working hypothesis is that this was a helper asset (hidden or low-profile) within an application’s infrastructure, used to carry HTML/JS for further abuse — e.g., injecting malicious code or enabling a man-in-the-middle step within a redirect chain. The `nham24` service itself is a legitimate shopping app popular mainly in Cambodia. It appears someone tried to leverage a resource on a trusted domain as an intermediary stage (**living off trusted domains**).

**Why call this email “deceptive”?** Because it’s assembled like a templated marketing blast: an official tone and a threat that you’ll lose Netflix access — but later in the email there’s no proof it’s genuine. The aim is to provoke a quick, reflexive click — exactly what the attacker wants.

<div align="center">
  <img src="/screenshots/mail5/redirect1.png" alt="HTML view" width="55%">
</div>
<p align="center"><em>Fig. 5 — nham24 login page</em></p>

To gather more detail, we open the email as **.txt** for deep inspection.

One of the first headers to check is **Return-Path**: `bounces+…@em872.ramnet.co.za`. The word “bounce” indicates a dedicated mailbox for nondelivery reports. Some bulk campaigns use **VERP** (Variable Envelope Return Path), e.g., `bounce+your.user=yourdomain.tld@esp.example`, so the system knows which recipient the bounce refers to.

In **From: Help Center <noreply_help.center764705@em872[.]ramnet[.]co[.]za>**, the sender pretends to be “Help Center,” but the real domain is `ramnet[.]co[.]za` — a common phishing trick to lend “support” credibility to a fake Netflix notice.

<div align="center">
  <img src="/screenshots/mail5/headers1.png" alt="HTML view" width="85%">
</div>
<p align="center"><em>Fig. 6 — Headers</em></p>

The **Received** chain shows the sender launched the mailing from their own machine/server (looks like Windows/EC2 in the cloud). The email wasn’t delivered directly; it went through **SendGrid** (a bulk-mail platform). Inside SendGrid it traversed internal hops (a `geopod-…` server, then an `outbound-mail` node) before being handed off to the recipient’s mail server.

**In short: sender’s host → SendGrid (internal relays) → recipient server.** This pattern is typical for commercial email-marketing accounts that make it easy to send templates impersonating well-known brands.

### Sender authentication

- **SPF — result: pass (em872[.]ramnet[.]co[.]za → 159[.]183[.]114[.]67)**  
  SPF pass only confirms the message was sent via authorized ESP infrastructure for **ramnet[.]co[.]za**. It is **not** proof of brand authenticity.

- **DKIM: pass, d=ramnet.co.za, s=s1, rsa-sha256**  
  The message is cryptographically signed by **ramnet.co.za** and verifies successfully. This confirms technical authenticity of **ramnet[.]co[.]za**, not of the brand shown in content.

- **DMARC: none**  
  **The sending domain publishes no restrictive DMARC policy.** As a result, there’s no enforced alignment between the **From:** domain and the domains validated by SPF/DKIM, which makes brand abuse easier.

<div align="center">
  <img src="/screenshots/mail5/headers2.png" alt="HTML view" width="85%">
</div>
<p align="center"><em>Fig. 7 — Authentication Results</em></p>

We also see **ARC (Authenticated Received Chain)** — a signed “evidence envelope” for SPF/DKIM/DMARC results that intermediaries attach so downstream hops can consider earlier authentication, even if later forwarding breaks SPF/DMARC. **Note:** ARC doesn’t guarantee authenticity; it’s just a signed history. You still need to assess brand/domain consistency, content, links, and other signals.

## Summary

- **Objective:** steal payment details/credentials via a fake subscription update (“Update payment method”).  
- **Attack type:** bulk **brand impersonation** (Netflix) + **credential/payment phishing** with **display-name spoofing** and a CTA outside the brand’s domain.  
- **Final assessment:** _**Phishing**_

---

### URL Analysis

**urlscan.io**
- `hxxps://pos[.]nham24[.]com/files/files/help.html` —  
  A record points to **165.22.60.15 (DigitalOcean, SG)** — a cloud VPS typical for short-lived campaigns. The path `/files/files/help.html` is odd (duplicated folder) — typical of hand-uploaded files or compromise.

<div align="center">
  <img src="/screenshots/mail5/urlscanio1.png" alt="HTML view" width="75%">
</div>

- `hxxps://em872.ramnet.co.za/` (sender subdomain) — **HTTP 400 during scan / DNS resolution error**.

**VirusTotal**
- `pos.nham24.com` — single detection (Fortinet: *Phishing*); most engines say clean. Page returns **404** (likely offline or cloaked). The odd subpath/subdomain suggests a short-lived URL used for fraud.

<div align="center">
  <img src="/screenshots/mail5/virustotal1.png" alt="HTML view" width="75%">
</div>

**WHOIS**
- `159.183.114.67` — points to an ESP block (**SendGrid**) with listed abuse/netops contacts — an official, legitimate IP range.

<div align="center">
  <img src="/screenshots/mail5/whois1.png" alt="HTML view" width="75%">
</div>

**MXToolbox**
- `wfbttnqp[.]outbound-mail[.]sendgrid[.]net` — no blacklist hits; warnings suggest configuration quirks of a major ESP (SendGrid), not definitive phishing proof.

<div align="center">
  <img src="/screenshots/mail5/mxtoolbox1.png" alt="HTML view" width="75%">
</div>

---

### Header Analysis

| Field                      | Value                                                                                | Note |
|---                         |---                                                                                   |---|
| `From`                     | `Help Center <noreply_help.center764705@em872.ramnet.co.za>`                         | Generic label + unrelated domain |
| `Reply-To`                 | —                                                                                    | No alternate reply channel |
| `Return-Path`             | `bounces+…@em872.ramnet.co.za`                                                       | Bounce channel in same domain as MAIL FROM |
| `Received` (last hop)     | `wfbttnqp.outbound-mail.sendgrid.net [159.183.114.67]` → recipient server            | SendGrid relay (mailing infrastructure) |
| **SPF**                    | `pass` for `em872.ramnet.co.za`                                                      | Authenticates **ramnet.co.za**, not the brand |
| **DKIM**                   | `pass` (d=ramnet.co.za; s=s1)                                                        | Signature of the technical sender’s domain |
| **DMARC**                  | `none`                                                                               | No DMARC policy (enables abuse) |

---

### IOC Table

| **Type**   | **Value**                                           | **Context**                                  | **Confidence** |
|---         |---                                                  |---                                           |---|
| **Domain** | `ramnet.co.za`                                      | Technical sender (MAIL FROM / DKIM d=)       | High |
| **Domain** | `em872.ramnet.co.za`                                | Subdomain used for send/bounces              | High |
| **IP**     | `159.183.114.67`                                    | `outbound-mail.sendgrid.net` relay           | Medium |
| **URL**    | `https://pos.nham24.com/files/files/help.html`      | Main CTA (“UPDATE NOW”)                      | High |
| **Domain** | `pos.nham24.com`                                    | CTA destination domain                       | High |
| **URL**    | (images) `https://cdn.nflximg.com/...` or proxy     | Visual branding used in content              | Medium |
| **Subject**| `Renew your subscription to avoid any delay on your service` | Time pressure / service loss        | Medium |
| **Phrase** | `Your account is on hold` / `Update your payment method` | Fear + urgency tactics                   | High |
