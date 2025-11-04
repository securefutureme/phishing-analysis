# Email 04 – Brand Impersonation 

While looking for more phishing examples, one message in my spam folder stood out. It differed from the others—it was fairly well crafted, at least in terms of page layout. However, things that might be missed at first glance become obvious after a moment’s look—for example, the subject line contains an emoji (something DPD does not do). Of course, someone who has never used DPD’s courier services might think this is a genuine DPD email. As we keep analyzing the message, more and more red flags start to appear:

<div align="center">
  <img src="/screenshots/mail4/message1.png" alt="Widok HTML" width="65%">
</div>
<p align="center"><em>Fig. 1 — Mail View</em></p>


- Sam układ strony przypomina trochę starszy, prawdziwy mail z DPD - jednak rok podpisu maila (2025) wskazuje na najnowszy email, jednak jak sprawdzimy w internecie, rozjeżdza się to z obecnymi, realnymi mailami DPD:

<div align="center">
  <img src="/screenshots/mail4/dpd.png" alt="Widok HTML" width="45%">
</div>
<p align="center"><em>Rys. 2 — Przykładowy mail DPD</em></p>

- DPD zwykkle kończy numer paczki z końcówką "U" (a przynajmniej obecnie w Polsce)
- Wyswietlany nadawca - **From:**„info@dpd[.]com[.]pl” nie pasuje do reszty - **<info@cheek-me[.]com>** — mamy tutaj do czynienia ze spoofingiem nazwy wyświetlanej.

<div align="center">
  <img src="/screenshots/mail4/message2.png" alt="Widok HTML" width="45%">
</div>
<p align="center"><em>Rys. 2 — Link z wiadomości</em></p>

Analizując jedyny dostępny link, który pojawia się po najechaniu na "Pokaż moje opcje". Pod tym właśnie CTA kryje się **prawdziwy cel kampanii**: hxxps://dpd-polska-15503340744219[.]sunnahbd[.]xyz/.

Całkiem sprytnie atakujący tutaj zamaskował nazwę marki w subdomenie i „numer przesyłki” wpleciony w host. Dla oka wygląda to jak oficjalny adres DPD, ale to tania sztuczka - prawdziwą domeną jest sunnahbd.xyz, kompletnie niezwiązana z DPD. Rzekomy numer paczki w subdomenie jest prawdopodobnie tokenem śledzącym/kampanijnym, który pozwala łączyć kliknięcia z adresami e-mail.

Cała strona przypomina użycie techniki **"clone phishingu"** - czyli skopiowany (niekoniecznie aktualny) szablon „DPD”, ale reszta zmodyfikowana i niekoniecznie tak samo zachowana.  Clone phishing polega na skopiowaniu konkretnej, wcześniej dostarczonej legalnej wiadomości. Tutaj jednak za dużo rzeczy się nam rozjeżdza, żeby to tak zaklasyfikować, ale ktoś ewidentnie używał orginalnego, starego szablonu maila DPD. Inną ważną czerwoną flagą jest logo osadzone jako **CID** (`cid:image001.png@…`) - zwykle firmy kurierskie używa swoich domen jako hosting obrazków. 

Następnie przechodzimy już do analizy nagłówków wiadomości:

<div align="center">
  <img src="/screenshots/mail4/headers.png" alt="Widok HTML" width="65%">
</div>
<p align="center"><em>Rys. 3 — Nagłówki wiadomości</em></p>

- **Return-Path:** `<info@cheek-me.com>` — podstawa na którą patrzymy - kanał zwrotny wskazuje **cheek-me[.]com** - czyli nie DPD.
- **DKIM:** podpis kryptograficzny jest prawidłowy, ale dla domeny cheek-me[.]com. Dowód, że ktoś kontroluje cheek-me[.]com i z tej domeny wysłał maila.
- **To / From:** w polach adresowych widzimy konstrukcję "info@dpd[.]com[.]pl" <info@cheek-me[.]com> – czyli wyświetlana nazwa udaje adres DPD, ale prawdziwy adres to znów info@cheek-me.com. Dodatkowo odbiorcą też jest ten sam e-mail. Taki układ jest charakterystyczny dla phishingu — ma udawać formalność, ale technicznie jest nielogiczny.
- **Łańcuch „Received”:** - możemy wywnioskować, że wiadomość została przyjęta z serwera **`sv16381.xserver.jp (85[.]131[.]207[.]122)`**, a źródłem nadania był host  **`p7cdba158[.]tokyff01[.]ap[.]so-net.ne.jp (124[.]219[.]161[.]88)`** (uwierzytelnione nadanie). To wygląda jak **zwykły hosting w Japonii**. Z DPD PL nie ma to nic wspólnego.

## Podsumowanie

- **Motyw:** podszycie pod DPD i przekierowanie do zewnętrznej strony z Bangladeszu;
- **Typ ataku:** Podszycie się pod markę razem ze spoofingiem nazwy wyświetlanej;
- **Ocena końcowa:** _**Phishing**_

### Analiza URL

**Do sprawdzenia:**
- **URL (CTA):** `hxxps://dpd-polska-15503340744219.sunnahbd[.]xyz/`
- **Domeny:** `sunnahbd[.]xyz`, `cheek-me[.]com`, `sv16381.xserver[.]jp`, `so-net[.]ne[.]jp`
- **IP:** `85.131.207.122` (sv16381.xserver.jp), `124.219.161.88` (so-net)

**urlscan.io**
- Szukaj: pełnego URL CTA i domeny `sunnahbd[.]xyz`
- Dodatkowo nazwa hosta: `sv16381.xserver[.]jp`

**VirusTotal**
- Sprawdź: `sunnahbd[.]xyz`, pełny URL CTA, `cheek-me[.]com`, IP `85.131.207.122` / `124.219.161.88`

**WHOIS**
- `sunnahbd[.]xyz`, `cheek-me[.]com` – daty rejestracji, rejestrator, kraj
- Reverse WHOIS/Passive DNS (jeśli masz dostęp) dla subdomen z prefiksem `dpd-polska-…`

**MXToolbox**
- `sunnahbd[.]xyz`, `cheek-me[.]com` – DNS/blacklisty, rekordy MX/SPF (jeśli istnieją)

### Analiza nagłówków

| Pole                      | Wartość                                                                           | Notatka |
|---                        |---                                                                                  |---|
| `From`                    | `"info@dpd.com.pl" <info@cheek-me[.]com>`                                          | Wyświetlana marka ≠ rzeczywisty adres. |
| `Reply-To`                | —                                                                                   | Brak alternatywnego kanału odpowiedzi w próbce. |
| `Return-Path`             | `<info@cheek-me[.]com>`                                                             | Często pokazuje realną domenę wysyłki. |
| `Received` (ostatni hop)  | `sv16381.xserver.jp (85.131.207.122)`                                               | Relay/hosting (Japonia). |
| `Received` (nadanie)      | `p7cdba158.tokyff01.ap.so-net.ne.jp (124.219.161.88)`; `with ESMTPSA`               | Uwierzytelnione nadanie z sieci dostępowej. |
| **SPF**                   | n/d w dostarczonym fragmencie                                                       | Brak `Authentication-Results` w próbce. |
| **DKIM**                  | `d=cheek-me.com; a=rsa-sha256;` / **good**                                          | Potwierdza domenę nadawcy, nie markę. |
| **DMARC**                 | n/d                                                                                 | Brak danych – nie widać ew. polityki. |

---

### Tabela IOC

| **Type** | **Value**                                               | **Context**                                  | **Confidence** |
|---|---|---|---|
| **Domain** | `cheek-me[.]com`                                      | Rzeczywisty nadawca (`From`/`Return-Path`)   | High |
| **Domain** | `sunnahbd[.]xyz`                                      | Domena landing page (CTA)                    | High |
| **IP**     | `85.131.207.122`                                      | `sv16381.xserver.jp` – relay/hosting         | Medium |
| **URL**    | `hxxps://dpd-polska-15503340744219.sunnahbd[.]xyz/`   | Główny link CTA                               | High |
| **Domain** | `sv16381.xserver[.]jp`                                | Serwer przyjmujący (ostatni hop)             | Medium |
| **URL**    | `mailto:` — brak (logo via CID)                       | Brak legalnych linków w domenie marki        | Medium |
