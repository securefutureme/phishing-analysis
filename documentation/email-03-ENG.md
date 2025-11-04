# Email 03 - Mass Promotional Spam

<div align="center">
  <img src="/screenshots/mail3/message1.png" alt="Widok HTML" width="65%">
</div>
<p align="center"><em>Fig. 1 — Mail View</em></p>

As in the previous phishing examples, when analyzing suspicious emails we first look for the most obvious “mismatches,” usually a disconnect between the **subject line** and the **message body**. Since this email came from the spam folder, I didn’t expect any advanced phishing techniques here. We immediately see the first red flag: **the subject doesn’t match the content — the subject claims to be a UPS delivery notice, while the body pushes an aggressive offer: “Buy 2 products, pay for only 1.”** This is a classic phishing pattern where the header pretends to be a logistics notification, but the content is a sales scheme or a prelude to phishing.

The longer we look at the email, the more signs we see that someone is trying to deceive us. **Another piece of evidence is the sender itself: “newsletter-NVH@xykTXwwtlvHiDsl[.]com,” allegedly writing on behalf of “Drmerritz.”** Serious companies don’t use random character strings in addresses or domains — they use consistent aliases (e.g., no-reply@google[.]com, news@updates[.]ubisoft[.]com) and sign emails with the same domain. This resembles messages used in ad campaigns sent from a randomly named newsletter that supposedly speaks on behalf of a brand.

Another common technique — here more of a marketing trick than strict phishing — is a subject line like “Don’t miss out!” In both phishing and marketing, senders play on **FOMO** (fear of missing out). It’s often the first thing that catches a user’s eye, and curiosity can beat caution, even subconsciously. In practice these messages shorten the time for reflection and increase the chance of a mindless click, especially when accompanied by a countdown timer, “limited spots,” or a large, high-contrast **CTA** (call to action) button.

<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/message2.png" width="600" alt="Widok maila (nagłówek)">
  </a>
</p>
<p align="center">
  <em>Fig. 2 — URL's from mail</em>
</p>

Analyzing only the message headers, we look at the **Received** chain:

- **from `swpi.edu.cn` (HELO `mta4.email.shopify.com`) `[167.172.61.220]` to the recipient’s server.**

The message arrived from a host identifying itself as `swpi.edu.cn` (a Chinese university domain), but in the HELO/EHLO command it presented as `mta4.email.shopify.com`.  
There’s too much inconsistency here—the host points to an academic institution, while **HELO** (an SMTP command the sending server uses at the start of the conversation to present its hostname) impersonates a commercial e-commerce platform. Such mismatch is unusual for legitimate sends (where PTR/HELO/SPF are typically aligned) and suggests misconfiguration or deliberate origin masking.

<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/message2.png" width="600" alt="Mail view (headers)">
  </a>
</p>
<p align="center">
  <em>Fig. 3 — Headers</em>
</p>

Additional header findings:

- **Sender field contains an address like `…@IjzSRhesiGzCDRg.pl.com.`** Such “glue” domains are often mass-registered, low-quality domains.
- **X-XX-DKIM-Status: no signature** — no cryptographic signature; the message lacks proof that the claimed domain actually sent it and that it wasn’t altered in transit.

Based on another clue, hovering over the **“Skorzystaj z okazji teraz”** (“Take the offer now”) button shows there isn’t even a redirect—  
the button is “empty,” with no link to a real site. That can irritate a regular user. And what does a user do when an email seems useless? They try to **unsubscribe**. Here comes the trick: the **only active link** in the email is **“Unsubscribe from the newsletter.”**

That link does not lead to a real unsubscribe page. Instead, through a tracking system it redirects—  
**`hxxps://dts.innovid.com/.../vclk?…&click=https://cdn.lasvegasusa.eu/...`**  
—to the final destination: **the casino site `cdn.lasvegasusa.eu`**. In practice:

- instead of unsubscribing — you confirm the mailbox is active,
- you land on a commercial/untrusted page unrelated to the supposed sender,
- you leave a footprint in campaign analytics, fueling further sends.

## Summary

- **Objective:** redirect to a campaign site (likely data/payment harvesting — cannot be verified due to an inactive campaign).
- **Attack type:** bulk phishing/scam impersonating a brand (UPS), with display-name/subject spoofing and a redirect chain built on links.
- **Final assessment:** _**Scam combined with phishing**_

### URL Analysis

##### urlscan.io

`cdn[.]lasvegasusa[.]eu`  
- One takeaway is that the `cdn.` subdomain may act as a CNAME to a provider (e.g., Cloudflare, Akamai, CloudFront). On the original **lasvegasusa** site I checked via DevTools → Network and indeed saw CDN usage. However, opening `cdn.lasvegasusa.eu` via urlscan.io currently leads nowhere. We can assume the redirect no longer works, or the ad campaign has changed/ended.

<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/urlscanio.png" width="600" alt="urlscan.io view">
  </a>
</p>

**`setblive.net`** and **`swpi.edu.cn`**  
- currently don’t resolve to any page. These may be fabricated or typo-lookalikes—e.g., `swpi.edu.cn` does not exist, while `sxpi.edu.cn` (or `swip.ac.cn`) does:

<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/typosquatting.png" width="600" alt="Typosquatting example">
  </a>
</p>

##### WHOIS

`167.172.61.220` — from the **Received** chain:  
WHOIS shows it belongs to the `167.172.0.0/16` block, assigned in the RIPE region. This range is associated with **DigitalOcean (AS14061)**, i.e., cloud/VPS, not Shopify. If the message claims `mta4.email.shopify.com` but the actual IP is `167.172.61.220` (DigitalOcean), we have a mismatch typical of brand impersonation (HELO/EHLO is easy to fake).

##### MXToolbox

MXToolbox for `swpi.edu.cn` supports the previous conclusion — `swpi.edu.cn` has no mail records; spoofed HELO/EHLO from `167.172.61.220`.

### Header Analysis

| Field                     | Value                                                                                                 | Note |
|---                        |---                                                                                                    |---|
| `From`                    | "Drmerritz" `<VHmYXKCAMiyxbbz@IKVUybJqdSUIJkm.com>`                                                   | Displayed brand ≠ sender domain |
| `Reply-To`                | `<newsletter-TSF@PkNLTsZYHCitHIJ.com>`                                                                | Different random domain than `From`/brand |
| `Return-Path`            | `<TfGxnDUaSOePORZ@IjzSRhesiGzCDRg.com>`                                                               | Often the real bounce channel; yet another random domain |
| `Received` (last hop)     | from **`swpi.edu.cn`** (HELO **`mta4.email.shopify.com`**) `[167.172.61.220]`                         | PTR/HELO mismatch (university vs. Shopify) and IP unrelated to the brand |
| **SPF**                   | n/a                                                                                                   | n/a |
| **DKIM**                  | `X-XX-DKIM-Status: no signature (id: n/a)`                                                            | No DKIM signature |
| **DMARC**                 | n/a                                                                                                   | n/a |

### IOC Table

| **Type** | **Value**                                                                                         | **Context**                                                    | **Confidence** |
|---       |---                                                                                                |---                                                              |---|
| **Domain** | `IKVUybJqdSUIJkm.com`                                                                           | Domain from `From` (random/non-brand)                            | High |
| **Domain** | `IjzSRhesiGzCDRg.com`                                                                           | `Return-Path` (bounce channel)                                   | High |
| **IP**     | `167.172.61.220`                                                                                | Source IP in `Received` (`swpi.edu.cn` / HELO Shopify)          | Medium |
| **URL**    | `https://dts.innovid.com/clktru/action/vclk?...&click=https://cdn.lasvegasusa.eu:443/...`       | Redirect/tracker → casino landing                             | High |
| **Domain** | `cdn.lasvegasusa.eu`                                                                            | Final landing (casino, unrelated to “UPS”)                  | Medium |
| **URL**    | `https://setblive.net/ZyjW455574401la14455EJOcukS`                                              | “Unsubscribe” leading through an external service           | High |
| **Subject**| 📦 UPS tracking update: [...]                                                                    | Brand/subject mismatch with sender domain                     | High |
| **Phrase** | “Don’t miss out!”                                                                               | Marketing pressure in content                                    | Low |
