# Email 04 - Podszycie się pod markę

Szukając innych przykładów phishingowych, w folderze spamowym jedna z nich przyciągnęła bardziej moją uwagę. Różniła się od innych - była całkiem dobrze zrobiona, przynajmniej jeżeli chodzi o układ strony. Jednak, to co "na szybko" może umknać, po kilku chwilach widać jak się przyjrzymy - w temacie mamy emoji w tytule (czego DPD nie robi). Oczywiście, ktoś, kto nigdy nie używał kuriera DPD może myśleć, że jest to prawdziwy email DPD. Analizując maila dalej, zaczynamy dostrzegać coraz więcej czerwonych flag:

<div align="center">
  <img src="/screenshots/mail4/message1.png" alt="Widok HTML" width="65%">
</div>
<p align="center"><em>Rys. 1 — Wiadomość</em></p>

- Sam układ strony przypomina trochę starszy, prawdziwy mail z DPD - jednak rok podpisu maila (2025) wskazuje na najnowszy email, jednak jak sprawdzimy w internecie, rozjeżdza się to z obecnymi, realnymi mailami DPD:

<div align="center">
  <img src="/screenshots/mail4/dpd.png" alt="Widok HTML" width="45%">
</div>
<p align="center"><em>Rys. 2 — Przykładowy mail DPD</em></p>

- DPD zwykkle kończy numer paczki z końcówką "U" (a przynajmniej obecnie w Polsce)
- Wyswietlany nadawca - **From:**`„info@dpd.com.pl”` nie pasuje do reszty - **`<info@cheek-me[.]com>`** — mamy tutaj do czynienia ze spoofingiem nazwy wyświetlanej.

<div align="center">
  <img src="/screenshots/mail4/message2.png" alt="Widok HTML" width="45%">
</div>
<p align="center"><em>Rys. 2 — Link z wiadomości</em></p>

Analizując jedyny dostępny link, który pojawia się po najechaniu na "Pokaż moje opcje". Pod tym właśnie CTA kryje się **prawdziwy cel kampanii**: `https://dpd-polska-15503340744219.sunnahbd.xyz/.`

Całkiem sprytnie atakujący tutaj zamaskował nazwę marki w subdomenie i „numer przesyłki” wpleciony w host. Dla oka wygląda to jak oficjalny adres DPD, ale to tania sztuczka - prawdziwą domeną jest sunnahbd.xyz, kompletnie niezwiązana z DPD. Rzekomy numer paczki w subdomenie jest prawdopodobnie tokenem śledzącym/kampanijnym, który pozwala łączyć kliknięcia z adresami e-mail.

Cała strona przypomina użycie techniki **"clone phishingu"** - czyli skopiowany (niekoniecznie aktualny) szablon „DPD”, ale reszta zmodyfikowana i niekoniecznie tak samo zachowana.  Clone phishing polega na skopiowaniu konkretnej, wcześniej dostarczonej legalnej wiadomości. Tutaj jednak za dużo rzeczy się nam rozjeżdza, żeby to tak zaklasyfikować, ale ktoś ewidentnie używał orginalnego, starego szablonu maila DPD. Inną ważną czerwoną flagą jest logo osadzone jako **CID** (`cid:image001.png@…`) - zwykle firmy kurierskie używa swoich domen jako hosting obrazków. 

Następnie przechodzimy już do analizy nagłówków wiadomości:

<div align="center">
  <img src="/screenshots/mail4/headers.png" alt="Widok HTML" width="65%">
</div>
<p align="center"><em>Rys. 3 — Nagłówki wiadomości</em></p>

- **Return-Path:** `<info@cheek-me.com>` — podstawa na którą patrzymy - kanał zwrotny wskazuje `**cheek-me.com**` - czyli nie DPD.
- **DKIM:** podpis kryptograficzny jest prawidłowy, ale dla domeny `cheek-me.com`. Dowód, że ktoś kontroluje `cheek-me.com` i z tej domeny wysłał maila.
- **To / From:** w polach adresowych widzimy konstrukcję `"info@dpd.com.pl"` `<info@cheek-me.com>` – czyli wyświetlana nazwa udaje adres DPD, ale prawdziwy adres to znów `info@cheek-me.com`. Dodatkowo odbiorcą też jest ten sam e-mail. Taki układ jest charakterystyczny dla phishingu — ma udawać formalność, ale technicznie jest nielogiczny.
- **Łańcuch „Received”:** - możemy wywnioskować, że wiadomość została przyjęta z serwera **`sv16381.xserver.jp (85.131.207.122)`**, a źródłem nadania był host  **`p7cdba158.tokyff01.ap.so-net.ne.jp (124.219.161.88)`** (uwierzytelnione nadanie). To wygląda jak **zwykły hosting w Japonii**. Z DPD PL nie ma to nic wspólnego.

## Podsumowanie

- **Motyw:** podszycie pod DPD i przekierowanie do zewnętrznej strony z Bangladeszu;
- **Typ ataku:** Podszycie się pod markę razem ze spoofingiem nazwy wyświetlanej;
- **Ocena końcowa:** _**Phishing**_

### Analiza URL

**urlscan.io**
- `https://dpd-polska-15503340744219.sunnahbd.xyz/` - błąd 400  – strona odrzuciła sposób, w jaki skaner się do niej odwołał. Może to znaczyć, że link działa tylko z konkretnym Referer, ciasteczkiem, jednorazowym parametrem albo po wcześniejszym kroku (np. POST do GET).

- `sunnahbd.xyz` - wnioskując po młodym wieku domeny, egzotycznym hostingu i treścią maila można wywnioskować to infrastrukturę do phishingu albo skompromitowany serwis WordPress użyty do hostowania klocków kampanii.
<div align="center">
  <img src="/screenshots/mail4/urlscanio1.png" alt="Widok HTML" width="85%">
</div>

- `cheek-me.com` - wygląda na zwykły serwis WordPress hostowany w XSERVER (JP). W mailu wykorzystano go jako kanał nadawczy. Albo jest to skompromitowany hosting użyty do wysyłki kampani albo jest to zewnętrznie użyty mailing w imieniu tej domeny (celowo i bez wiedzy właściciela).
<div align="center">
  <img src="/screenshots/mail4/urlscanio2.png" alt="Widok HTML" width="85%">
</div>

**VirusTotal**
- `sunnahbd.xyz` - Virustotal nie wykrywa niczego podejrzanego, oprócz **alphaMountain.ai**, która oflagowała stronę jako podejrzaną. 
<div align="center">
  <img src="/screenshots/mail4/virustotal.png" alt="Widok HTML" width="75%">
</div>

**WHOIS**

- `85.131.207.122` - XSERVER (JP) – ten sam operator, na którym stoi domena nadawcy `cheek-me.]om`. IP ma rDNS `sv16381.xserver.jp`, typowe dla serwerów pocztowowych. Zapewne po prostu posłużyło jako infrastruktura wysyłkowa kampanii.
<div align="center">
  <img src="/screenshots/mail4/whois1.png" alt="Widok HTML" width="75%">
</div>

- `124.219.161.88` - Z tego IP nastąpiło uwierzytelnione nadanie (ESMTPSA) na serwer `sv16381.xserver.jp`, więc ktoś zalogował się do hostingu XSERVER i wysłał wiadomość.
<div align="center">
  <img src="/screenshots/mail4/whois2.png" alt="Widok HTML" width="75%">
</div>

**MXToolbox**
- `cheek-me.com` – Wniosek po sprawdzeniu tego w mxtoolboxie jest jeden - nadawca faktycznie kontroluje `cheek-me.com`, lecz zabezpieczenia poczty są słabe, a i tak nie ma to związku z DPD.
<div align="center">
  <img src="/screenshots/mail4/mxtoolbox1.png" alt="Widok HTML" width="95%">
</div>


### Analiza nagłówków

| Pole                      | Wartość                                                                           | Notatka |
|---                        |---                                                                                  |---|
| `From`                    | `"info@dpd.com.pl" <info@cheek-me.com>`                                           | Wyświetlana marka to nie rzeczywisty adres |
| `Reply-To`                | —                                                                                   | Brak alternatywnego kanału odpowiedzi w próbce |
| `Return-Path`             | `<info@cheek-me.com>`                                                             | Często pokazuje realną domenę wysyłki |
| `Received` (ostatni hop)  | `sv16381.xserver.jp (85.131.207.122)`                                               | hosting wysłanego maila pochodzi z Japoni |
| `Received` (nadanie)      | `p7cdba158.tokyff01.ap.so-net.ne.jp (124.219.161.88)`; `with ESMTPSA`               | Uwierzytelnione nadanie z sieci dostępowej |
| **SPF**                   | n/d w dostarczonym fragmencie                                                       | n/d |
| **DKIM**                  | `d=cheek-me.com; a=rsa-sha256;` / **good**                                          | Potwierdza domenę nadawcy, nie markę |
| **DMARC**                 | n/d                                                                                 | n/d |

---

### Tabela wskaźnika IOC

| **Typ**   | **Wartość**                                                                                           | **Kontekst**                                          | **Poziom wiarygodności wskaźnika** |
|---|---|---|---|
| **Domena** | `cheek-me[.]com`                                      | Rzeczywisty nadawca                         | Wysoki |
| **Domena** | `sunnahbd[.]xyz`                                      | Domena głownego przycisku akcji                  | Wysoki |
| **IP**     | `85.131.207.122`                                      | `sv16381.xserver.jp` – hosting mailowy          | Średni |
| **URL**    | `https://dpd-polska-15503340744219.sunnahbd.xyz/`     | Główny link przycisku akcji (jedyny)            | Wysoki |
| **Domena** | `sv16381.xserver.jp`                                  | Serwer przyjmujący (ostatni hop)             | Średni |
