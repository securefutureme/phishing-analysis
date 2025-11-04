
<details>
  <summary><b>Polski — kliknij, by rozwinąć</b></summary>

<p>

# Projekt - Analiza maila phishingowego 

### Wstęp
Repozytorium to prezentuje oparte na przykładach analizy wiadomości phishingowych — zarówno autentycznych, jak i zmyślonych. Celem jest rozwój kompetencji w zakresie identyfikacji sygnałów ostrzegawczych oraz weryfikacji autentyczności komunikatów. Umiejętność rozpoznawania czerwonych flag i weryfikacji autentyczności jest kluczowa, ponieważ:

- Użytkownicy końcowi zwiększają odporność na socjotechnikę, redukując prawdopodobieństwo kliknięcia w złośliwe odnośniki już na najwcześniejszym etapie incydentu.
- Helpdesk/SOC usprawnia proces triage oraz eskalację (blokady domen/IP, IOC do SIEM), co skraca czas reakcji.
- Admnistratorom/cybersecurity daje podstawę do twardych kontroli (DMARC, filtrowanie, makra, zgłaszanie jednym kliknięciem).
- Kadry i menedżerowie otrzymują klarowne rekomendacje ograniczające ekspozycję na ryzyka oraz koszty (fraud, przestoje, kary).

**Adresuję moje repo do:**
- użytkowników – jak rozpoznać phishing i czego nie robić;
- SOC/IT – jako przykładowy sposób opisywania analizy phishingu;
- organizacji – w celach szkoleniowych;

#### Spis analiz

| Nazwa analizy | Link |
|---|---|
| Masowy phishing z narzuceniem presji czasu | [Mail 1](documentation/email-01-PL.md) |
| Wyłudzenie danych logowania | [Mail 2](documentation/email-02-PL.md) |
| Masowa kampania reklamowa | [Mail 3](documentation/email-03-PL.md) |
| Podszycie się pod markę | [Mail 4](documentation/email-04-PL.md) |
| Phishing "zwodniczy", jako technika phishingu | [Mail 5](documentation/email-05-PL.md) |

---

### Co to jest phishing?
Phishing jest to metoda oszustwa opierająca się na **„podszywaniu”** się pod inną osobę albo instytucję, próbując w ten sposób **wyłudzić poufne dane, zainfekować komputer czy inne nieprzyjazne działania.**

#### Przykłady phishingu:

- **Spear phishing** — celowany mail do konkretnej osoby/zespołu, z detalami z OSINT („Cześć Kasiu, z HR…”) i linkiem/załącznikiem do kradzieży haseł lub malware.
- **Phishing zwodniczy (deceptive phishing)** -  odmiana albo technika phishingu e-mailowego, w której przestępcy używają podstępnych technik, często używając bardzo podobnych maili prawdziwych marek, by skłonić ofiarę do kliknięcia linku i zainfekowania urządzenia.
- **Vishing** — voice phishing; oszustwo telefoniczne, w którym przestępcy podszywają się pod zaufane instytucje (np. banki, policję) lub osoby, aby wyłudzić poufne dane.
- **Smishing** — SMS phishing; SMS z linkiem do fałszywej wiadomości, np. bramki płatności/śledzenia paczki lub „dopłaty 3,99 zł”.
- **Whaling / CEO Fraud** — atak na kadrę (CEO/CFO); mail „od prezesa” z prośbą o pilny przelew lub poufne dane.
- **BEC — Business Email Compromise** — przejęta prawdziwa skrzynka firmy/partnera; zamiana numerów kont na fakturach, „aktualizacja danych płatności” itp.
- **SEO poisoning / Search Engine Phishing** — złośliwe strony wypchnięte wysoko w Google (często reklamy); użytkownik klika „pobierz Teams/Bank” i trafia na podróbkę.
- **Clone phishing** — imitowanie legalnych wiadomości e-mail od zaufanych firm w celu nakłonienia ofiar do ujawnienia poufnych informacji.
- **QR phishing (QRishing)** — złośliwy kod QR w mailu/ulotce prowadzący do fałszywej strony logowania lub strony atakującego.
- **Consent/OAuth phishing** — zamiast hasła prosi o „Zgódź się na dostęp” dla rzekomej aplikacji; w rzeczywistości oddajesz tokeny do konta.
- **MFA fatigue** — technika „bombardowania” push'ami MFA do ofiary, aż zaakceptuje „przypadkowo”.
- **Tech support / angler phishing** — fałszywy czat/ogłoszenie „pomocy technicznej” (często w social media), namawia do instalacji narzędzi zdalnych.
- **Pharming/Wi-Fi captive hijack** — przekierowanie na fałszywą stronę (np. manipulacja DNS/hosts), identyczną jak legalna, by wyłudzić dane.
- **Watering hole** — zainfekowanie często odwiedzanej strony (np. branżowy portal), by złowić loginy/zainfekować malware.

---

**Workflow:**  
VM → eksport maili phishingowych → eksport treści/nagłówków → analiza wedle schematu → toole analizujące adresy IP i linki → **Tabela IOC**

**Słowniczek:**
- **ARC (Authenticated Received Chain)** – łańcuch podpisów dodawanych przez kolejne bramy pocztowe, pozwala zachować wyniki uwierzytelniania po przekazaniu/forwardzie.
- **ASN (Autonomous System Number)** – numer systemu autonomicznego w routingu Internetu; pomaga kojarzyć IP z operatorem/hostingiem.
- **Authentication-Results** – nagłówek raportujący wyniki SPF/DKIM/DMARC u odbiorcy.
- **Base64** – sposób kodowania binariów (np. obrazków) w treści e-mail (MIME).
- **Brand impersonation** – podszywanie się pod markę (nazwa, logo, styl).
- **CTA (Call to Action)** – element w treści zachęcający do kliknięcia (np. „Pokaż moje opcje”).
- **Credential phishing** – wyłudzanie poświadczeń (login/hasło) zwykle przez fałszywe strony logowania.
- **DKIM (DomainKeys Identified Mail)** – podpis kryptograficzny treści wiadomości, weryfikowany kluczem z DNS.
- **DMARC (Domain-based Message Authentication, Reporting and Conformance)** – polityka nadawcy określająca jak odbiorca ma traktować maile niezgodne ze SPF/DKIM (np. p=none/quarantine/reject).
- **Display-name spoofing** – nadużycie wyświetlanej nazwy nadawcy (np. „DPD Support”) przy innym, faktycznym adresie.
- **Domain mismatch / brand-domain mismatch** – rozjazd marki w treści z faktyczną domeną nadawcy lub linku.
- **Envelope-From / Mail From** – techniczny adres nadawcy używany w warstwie SMTP (widoczny np. w Return-Path), może różnić się od From:.
- **ESMTP / ESMTPSA** – rozszerzony SMTP; ESMTPSA to uwierzytelnione TLS-owe nadanie przez klienta do serwera.
- **Forwarder / listserwer** – pośrednik przekazujący wiadomości dalej; może modyfikować nagłówki i wyniki SPF.
- **Gateway / MTA (Mail Transfer Agent)** – serwer pośredniczący w przyjmowaniu i dalszym wysyłaniu poczty (np. Postfix).
- **Header (nagłówek)** – metadane e-mail (From, To, Subject, Received itd.).
- **HTML-part / text-part** – części wieloczęściowego (MIME) maila: HTML i zwykły tekst (multipart/alternative).
- **IOC (Indicator of Compromise)** – wskaźnik naruszenia: domena, IP, URL, fraza, temat itd.
- **IP reputation / blacklist** – ocena zaufania do adresu IP, użyteczna w filtrowaniu spamu.
- **Link wrapper (np. www.google.com/url?q=…)** – pośredni adres maskujący właściwy cel linku.
- **MIME (Multipurpose Internet Mail Extensions)** – standard opakowania treści e-mail (format, załączniki).
- **MTA-STS / TLS **– mechanizmy wymuszania bezpiecznego dostarczania poczty; TLS szyfruje transport.
- **MX (Mail eXchanger)** – rekord DNS wskazujący serwer przyjmujący pocztę dla domeny.
- **Outlook/Word VML** – specyficzne znaczniki MS Office w HTML e-mail; częsty znak „ręcznie klejonych” szablonów.
- **Pipeline (łańcuch dostarczenia)** – kolejność serwerów z Received: (host źródłowy → bramy pośrednie → serwer odbiorcy).
- **PIXEL / tracking pixel** – niewidoczny obraz (często z unikalnym ID) do śledzenia otwarć wiadomości.
- **Quarantine / reject (DMARC)** – akcje zalecane odbiorcy wobec niezgodnych wiadomości (kwarantanna lub odrzucenie).
- **Received** – nagłówek dodawany przez każdy serwer po drodze; służy do rekonstrukcji trasy maila.
- **Return-Path** – adres zwrotny (bounce), zwykle odpowiada envelope-from; często ujawnia rzeczywistego nadawcę.
- **Sandbox / podgląd** – bezpieczne środowisko do analizy treści wiadomości i linków.
- **SendGrid / ESP (Email Service Provider)** – komercyjna platforma wysyłkowa; często nadużywana w kampaniach.
- **SHA-1 / SHA-256** – funkcje skrótu; w DKIM zalecany jest SHA-256 (SHA-1 uznawany za przestarzały).
- **Shared hosting / VPS** – współdzielony hosting vs. prywatny serwer wirtualny; często tło kampanii phishingowych.
- **SOC (Security Operations Center)** – zespół operacyjny bezpieczeństwa, reagujący na incydenty.
- **SPF (Sender Policy Framework)** – mechanizm sprawdzania, czy dany IP może wysyłać pocztę dla domeny.
- **Subject obfuscation** – zaciemnianie tytułu (emoji, kodowanie =?UTF-8?B?...?=) w celu ominięcia filtrów/uwagi.
- **Tenant impersonation** – podszywanie się pod dzierżawcę/usługę (np. portal firmowy, skrzynka) bez użycia jego domeny.
- **Threat actor / atakujący** – podmiot prowadzący kampanię.
- **TTL (Time To Live)** – czas ważności odpowiedzi DNS; krótki TTL ułatwia szybkie rotacje.
- **URL / Effective URL** – adres docelowy po przekierowaniach; istotny w analizie celu kampanii.
- **urlscan.io / VirusTotal / WHOIS / MXToolbox** – narzędzia OSINT do analizy stron, reputacji, rejestracji, konfiguracji poczty.
- **VERP / track ID** – unikalny identyfikator w adresie/ścieżce do śledzenia kampanii i odbiorców.
- **VPS/host źródłowy** – komputer kliencki lub serwer, z którego nastąpiło uwierzytelnione nadanie (często widoczne w najniższym Received).

---

### INFO
- **Niektóre dane, jak mój adres mailowy i inne zostały usunięte, z powodów prywatnych.**
- **Ten projekt jest ćwiczeniem obrazującym moje umiejętności analizy maila phishingowego po odbyciu kursu Szkoła Security.**

---

</p>

</details>

<details>
  <summary><b>English — click to expand</b></summary>

<p>

# Phishing mail analysis project

### Introduction
This repository presents example-driven analyses of phishing emails—both real and simulated. The goal is to build skills in spotting warning signs and verifying message authenticity. Recognizing red flags and validating legitimacy is critical because:

- End users become more resilient to social engineering, reducing the likelihood of clicking malicious links at the earliest stage of an incident.
- Helpdesk/SOC teams streamline triage and escalation (domain/IP blocks, IOCs to SIEM), shortening response times.
- Administrators/cybersecurity teams gain a foundation for hard controls (DMARC, filtering, macros, one-click reporting).
- Leaders and managers receive clear recommendations that reduce risk exposure and costs (fraud, downtime, penalties).

**This repo is intended for:**
- **Users** — how to recognize phishing and what **not** to do.
- **SOC/IT** — an example structure for documenting phishing analysis.
- **Organizations** — training purposes.

| Analysis name | ENG |
|---|---|
| Mass-phishing using urgency tactics | [Mail 1](documentation/email-01-ENG.md) |
| Credential Harvesting | [Mail 2](documentation/email-02-ENG.md) |
| Mass Promotional Spam | [Mail 3](documentation/email-03-ENG.md) |
| Brand Impersonation  | [Mail 4](documentation/email-04-ENG.md) |
| Deceptive phishing | [Mail 5](documentation/email-05-ENG.md) |

---

### What is phishing?
Phishing is a social-engineering technique where an attacker **impersonates** a person or organization to **steal sensitive data, infect devices, or perform other harmful actions.**

#### Examples of phishing:

- **Spear phishing** — a targeted email to a specific person/team using OSINT details (“Hi Kasia from HR…”) with a link/attachment to steal credentials or deliver malware.
- **Deceptive phishing** -  is a type of email-phishing technique in which attackers use deceptive tactics - often sending emails that closely imitate legitimate brands to trick victims into clicking a link and infecting their device.
- **Vishing** — voice phishing; phone scams impersonating trusted institutions (banks, police) or individuals to extract personal data.
- **Smishing** — SMS phishing; a text with a link to a fake page, e.g., payment/tracking gateway or “extra fee: 3.99 PLN”.
- **Whaling / CEO Fraud** — targeting executives (CEO/CFO); “from the boss” requesting urgent transfers or sensitive data.
- **BEC — Business Email Compromise** — a compromised legitimate mailbox of a company/partner; bank accounts swapped on invoices, “payment details update”, etc.
- **SEO poisoning / Search Engine Phishing** — malicious sites boosted in search (often via ads); users click “download Teams/Bank” and land on a clone.
- **Clone phishing** — replicating legitimate emails from trusted brands to coax victims into revealing sensitive info.
- **QR phishing (QRishing)** — a malicious QR code in an email/flyer leading to a fake login page or attacker site.
- **Consent/OAuth phishing** — instead of a password, asks you to “Grant access” to a purported app; in reality you hand over tokens.
- **MFA fatigue** — “bombarding” a victim with MFA push prompts until they accept one “by accident”.
- **Tech support / angler phishing** — fake “support” chats/ads (often on social media) prompting installation of remote tools.
- **Pharming/Wi-Fi captive hijack** — redirecting to a fake site (via DNS/hosts tampering) that looks identical to the real one to harvest credentials.
- **Watering hole** — compromising a frequently visited site (e.g., industry portal) to capture logins or plant malware.

---

**Workflow:**  
VM --> export phishing emails --> export content/headers --> analysis by schema --> tools to assess IPs and links --> **IOC Table**

**Glossary:**  
- **ARC (Authenticated Received Chain)** – a chain of signatures added by mail gateways to preserve authentication results across forwarding.
- **ASN (Autonomous System Number)** – an Internet routing identifier that links IP ranges to an operator/hosting provider.
- **Authentication-Results** – a header reporting the recipient’s SPF/DKIM/DMARC results.
- **Base64** – a way to encode binaries (e.g., images) inside email (MIME).
- **Brand impersonation** – pretending to be a brand (name, logo, style).
- **CTA (Call to Action)** – an element prompting a click (e.g., “Show my options”).
- **Credential phishing** – stealing credentials (login/password), usually via fake login pages.
- **DKIM (DomainKeys Identified Mail)** – a cryptographic signature of the message, verified with a DNS key.
- **DMARC (Domain-based Message Authentication, Reporting and Conformance)** – a sender policy telling receivers how to treat mail failing SPF/DKIM (e.g., p=none/quarantine/reject).
- **Display-name spoofing** – abusing the visible sender name (e.g., “DPD Support”) while using a different actual address.
- **Domain mismatch / brand-domain mismatch** – brand shown in content doesn’t match the real sender or link domain.
- **Envelope-From / Mail From** – the SMTP-layer sender address (shows in Return-Path), can differ from `From:`.
- **ESMTP / ESMTPSA** – Extended SMTP; ESMTPSA is authenticated TLS submission from client to server.
- **Forwarder / list server** – an intermediary that forwards mail and may alter headers/auth results.
- **Gateway / MTA (Mail Transfer Agent)** – a mail relay server (e.g., Postfix).
- **Header** – email metadata (From, To, Subject, Received, etc.).
- **HTML-part / text-part** – the HTML and plain-text parts of a multipart (MIME) email.
- **IOC (Indicator of Compromise)** – an indicator such as a domain, IP, URL, phrase, or subject.
- **IP reputation / blacklist** – trust/abuse rating for an IP; used in spam filtering.
- **Link wrapper (e.g., `www.google.com/url?q=…`)** – an intermediate URL that masks the real destination.
- **MIME (Multipurpose Internet Mail Extensions)** – the standard for packaging email content/attachments.
- **MTA-STS / TLS** – mechanisms to enforce secure mail delivery; TLS encrypts transport.
- **MX (Mail eXchanger)** – DNS record pointing to a domain’s inbound mail server.
- **Outlook/Word VML** – Microsoft-specific HTML tags; often seen in hand-built email templates.
- **Pipeline (delivery chain)** – the order of servers in `Received:` (source host → intermediaries → recipient server).
- **PIXEL / tracking pixel (1×1)** – an invisible image (often with a unique ID) to track opens.
- **Quarantine / reject (DMARC)** – receiver actions recommended for noncompliant mail.
- **Received** – a header added by each server along the path; used to reconstruct the route.
- **Return-Path** – bounce address (envelope-from); often reveals the true sender domain.
- **Sandbox / preview** – a safe environment to inspect emails and links.
- **SendGrid / ESP (Email Service Provider)** – a commercial bulk-mail platform; often abused in campaigns.
- **SHA-1 / SHA-256** – hash functions; DKIM should use SHA-256 (SHA-1 is considered obsolete).
- **Shared hosting / VPS** – shared web hosting vs. a virtual private server; common for phishing infrastructure.
- **SOC (Security Operations Center)** – a team that handles security monitoring and incidents.
- **SPF (Sender Policy Framework)** – checks whether an IP is authorized to send mail for a domain.
- **Subject obfuscation** – disguising the subject (emoji, `=?UTF-8?B?...?=`) to dodge filters or attention.
- **Tenant impersonation** – posing as a tenant/service (e.g., company portal/mailbox) without using its domain.
- **Threat actor / attacker** – the entity running the campaign.
- **TTL (Time To Live)** – DNS record lifetime; short TTL enables fast rotations.
- **URL / Effective URL** – the final destination after redirects; key for analyzing the campaign’s goal.
- **urlscan.io / VirusTotal / WHOIS / MXToolbox** – OSINT tools for page analysis, reputation, registration, and mail config.
- **VERP / track ID** – unique identifier in an address/path for tracking a campaign and recipients.
- **VPS / source host** – the client or server from which the message was submitted (often seen in the lowest `Received`).
---

### INFO
- **Some data (e.g., my email address) has been removed for privacy reasons.**
- **This project demonstrates my phishing-email analysis skills after completing the “Szkoła Security” course.**

---
</p>

</details>

