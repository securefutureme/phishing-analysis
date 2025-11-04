# Email 05 - Phishing "zwodniczy", jako technika phishingu

**************--------------------------------------------------------mail pochodzący z monkey.org-----------------------------------------------------------**************

O tę próbkę zahaczyłem, przeglądając jeden z najbardziej znanych publicznych zbiorów phishingu — Jose Nazario phishing corpus. To świetny materiał treningowy dla SOC - setki realnych kampanii, które pozwalają ćwiczyć umiejętności analityczne bez ryzyka dotykania żywej infrastruktury ofiary. Wiadomość, jak zwykle, otworzyłem w izolowanym środowisku. Na pierwszy rzut oka przywołuje dobrze znany motyw **odnowienia subskrypcji**:

<div align="center">
  <img src="/screenshots/mail5/message1.png" alt="Widok HTML" width="35%">
</div>
<p align="center"><em>Rys. 1 — Wiadomość</em></p>

Mamy tutaj do czynienia z phishingiem podszywającym się pod markę "Netflix". Czemu od raz bym zaklasyifkował to jako phishing? Ponieważ pierwsza czerwona flaga rzuca się od razu w oczy:

<div align="center">
  <img src="/screenshots/mail5/header1.png" alt="Widok HTML" width="55%">
</div>
<p align="center"><em>Rys. 2 — Nagłówki</em></p>

Mamy rozjechanie się treści maila z adresem odbiorcy. Netflix nie przesyła maili o odnowienie subskrybcji z adresów mailowych nie zakończonych swoją domeną. Tutaj podam przykład ze swojego prywatnego maila:

<div align="center">
  <img src="/screenshots/mail5/orginalnetfl.png" alt="Widok HTML" width="55%">
</div>
<p align="center"><em>Rys. 3 — Nagłówek prawdziwego maila Netflix</em></p>

Ostatecznym dowodem (od strony patrzenia użytkownika), jest tutaj link z przycisku CTA - "Update now" - który ewidentnie **nie** prowadzi do żadnej domeny Netflix:
<div align="center">
  <img src="/screenshots/mail5/message3.png" alt="Widok HTML" width="55%">
</div>
<p align="center"><em>Rys. 4 — Nagłówek prawdziwego maila Netflix</em></p>

Ondnośnik aktualnie nie zwraca treści (serwer odpowiada błędem 404): **`https://pos.nham24.com/files/files/help.html`**. Hipoteza zakłada, że był to zasób pomocniczy (ukryty lub mało eksponowany) w infrastrukturze aplikacji, wykorzystany jako nośnik treści HTML/JS do dalszego nadużycia – np. wstrzyknięcia złośliwego kodu lub przeprowadzenia ataku Man-in-the-Middle (MitM) w ramach łańcucha przekierowań. Sama usługa nham24 działa i jest legalną aplikacją zakupową, popularną głównie w Kambodży. Można wnioskować, że ktoś próbował wykorzystać zasób w zaufanej domenie jako etap pośredni kampanii (tzw. living-off-trusted-domains).

**Dlaczego nazwałem tego maila "zwodniczym"?** Ponieważ cała wiadmość jest sklejona jak kampania marketingowa z szablonu. Narzucenie oficjalnego tonu, groźba, że nie będziesz mógł oglądać Netflixa, jednak w dalszej części maila jest brak dowodów, że to prawdziwy email. W efekcie odbiorca ma odruch kliknięcia, żeby szybko zobaczyć co się dzieje, choć to właśnie o ten bezrefleksyjny klik chodzi atakującemu.

<div align="center">
  <img src="/screenshots/mail5/redirect1.png" alt="Widok HTML" width="55%">
</div>
<p align="center"><em>Rys. 5 — Strona logowania nham24</em></p>

By zebrać jeszcze więcej informacji na temat tego maila phishingowego, otworzymy email w pliku .txt, by dogłebnie go przeanalizować. 

Jeden z pierwszych nagłówków, na który patrzymy to **Return-Path** `bounces+…@em872.ramnet.co.za`. Słowo „bounce” sygnalizuje, że to dedykowana skrzynka do obsługi odbić i raportów. Niektóre kampanie masówek, używają VERP (Variable Envelope Return Path) - `**np:bounce+twoj.user=twojadomena.pl@esp.example.**` Dzięki temu system wie, którego odbiorcy dotyczyło odbicie. 

W nagłówku **From: Help Center `<noreply_help.center764705@em872.ramnet.co.za>**` widzimy jak nadawca podszywa się pod „Help Center”, ale realna domena to `ramnet.co.za.` Częste zagranie w phishingu, by nadać "realności" mailowi, który ma pochodzić rzekomo od supportu Netflix.

<div align="center">
  <img src="/screenshots/mail5/headers1.png" alt="Widok HTML" width="85%">
</div>
<p align="center"><em>Rys. 6 — Nagłówki </em></p>

Łańcuch **Received** pokazuje nam jasno, że ktoś uruchomił wysyłkę ze swojego komputera/serwera (wygląda na Windows/EC2 w chmurze). Ten mail nie leciał prosto do odbiorcy, tylko trafił do SendGrid – platforma do masowej wysyłki. W środku SendGrid wiadomość przechodziła przez ich „przystanki” techniczne (serwer o nazwie geopod-…, potem węzeł outbound-mail) i dopiero stamtąd była przekazana do serwera pocztowego adresata.

**W skrócie: komputer nadawcy → SendGrid (wewnętrzne serwery) → serwer odbiorcy.** Taki łańcuch jest charakterystyczny dla komercyjnych kont do e-mail marketingu, z których łatwo rozsyłać gotowe szablony podszywające się pod znane marki.

### Autoryzacja nadawcy 

- **SPF — wynik: pass (`em872.ramnet.co.za` -> `159.183.114.67)`** - SPF pass potwierdza jedynie, że wiadomość wysłano przez uprawnioną infrastrukturę ESP dla domeny `ramnet.co.za`. Nie stanowi dowodu autentyczności maila.

- **DKIM: pass, d=`ramnet.co.za`, s=s1, algorytm rsa-sha256**   - Wiadomość została kryptograficznie podpisana przez domenę `ramnet.co.za` i weryfikacja podpisu zakończyła się powodzeniem. Potwierdzenie DKIM nie dowodzi jednak zgodności z marką widoczną w treści — jedynie autentyczność techniczną domeny `ramnet.co.za`
- **DMARC: none**- **Domena nadawcza nie publikuje restrykcyjnej polityki DMARC.** Skutkiem jest brak wymuszenia wyrównania identyfikatorów (alignment) między domeną w nagłówku From: a domenami używanymi przez SPF/DKIM. Taki stan zwiększa ryzyko nadużyć wizerunkowych.

<div align="center">
  <img src="/screenshots/mail5/headers2.png" alt="Widok HTML" width="85%">
</div>
<p align="center"><em>Rys. 7 — Authentication Results </em></p>

**Mamy tu także ARC (Authenticated Received Chain), czyli coś w rodzaju „koperty dowodowej” dla wyników SPF/DKIM/DMARC**, którą pośredniczące serwery dopinają do wiadomości, aby zachować historię uwierzytelnienia podczas dalszego przekazywania (forwardy, listy dyskusyjne, bramy). Ponieważ takie przekazania często „psują” SPF/DMARC (zmienia się IP nadawcy, dopisywane są prefiksy do tematu, wstawiane stopki itd.), ARC pozwala kolejnym serwerom brać pod uwagę podpisane wyniki z wcześniejszego hopa — nawet jeśli na ich etapie SPF/DMARC już się nie zgadzają. Innymi słowy: lokalnie testy mogą wyjść źle, ale podpisany łańcuch mówi, że „u poprzednika było OK”, więc MTA może to uwzględnić w ocenie.

**Ważna uwaga: ARC nie gwarantuje autentyczności wiadomości**— to tylko podpisana historia wyników. Nadal trzeba oceniać kontekst: zgodność marki z domeną, treść, linki i pozostałe sygnały.

## Podsumowanie

- **Motyw:** wyłudzenie danych płatniczych / poświadczeń przez fałszywą aktualizację subskrypcji („Update payment method”).
- **Typ ataku:** masowy **brand impersonation** (Netflix) + **credential/payment phishing** z **display-name spoofing** i CTA prowadzącym poza domenę marki.
- **Ocena końcowa:** _**Phishing**_

---

### Analiza URL

**urlscan.io**
- `https://pos.nham24.com/files/files/help.html` -
Adres A -> 165.22.60.15 (DigitalOcean, SG) wskazuje na VPS w chmurze; to typowa infrastruktura „jednorazówek” (łatwo założyć, łatwo porzucić). Ścieżka /files/files/help.html wygląda nienaturalnie (duplikat katalogu) — typowe dla ręcznie wrzuconych plików lub kompromitacji.

<div align="center">
  <img src="/screenshots/mail5/urlscanio1.png" alt="Widok HTML" width="75%">
</div>

- `https://em872.ramnet.co.za/` (subdomena nadawcy) - **HTTP 400 Error przy skanie - nie można rozwiązać domeny (DNS error)**

**VirusTotal**
- `pos.nham24.com` - tylko jedna detekcja - (Fortinet: Phishing), przy czym większość silników oznacza go jako czysty. Strona zwraca błąd 404, więc prawdopodobnie jest już wyłączona lub ukryta. Nietypowa nazwa podstrony i poddomena sugerują, że mógł to być krótko działający adres użyty w oszustwie.

<div align="center">
  <img src="/screenshots/mail5/virustotal1.png" alt="Widok HTML" width="75%">
</div>


**WHOIS**

- `[159.183.114.67)` - Wskazuje na infrastrukturę dostawcy mailingu (ESP) z podanymi kontaktami abuse i netops—czyli oficjalny, legalny blok adresów SendGrid
<div align="center">
  <img src="/screenshots/mail5/whois1.png" alt="Widok HTML" width="75%">
</div>


**MXToolbox**
- `wfbttnqp.outbound-mail.sendgrid.net` – Brak wpisów na czarnych listach, więc domena nie jest oznaczona jako złośliwa. Wskazane alerty sugerują raczej niespójności konfiguracyjne dużego ESP (SendGrid) niż jednoznaczny dowód phishingu.
<div align="center">
  <img src="/screenshots/mail5/mxtoolbox1.png" alt="Widok HTML" width="75%">
</div>


---

### Analiza nagłówków

| Pole                      | Wartość                                                                                 | Notatka |
|---                        |---                                                                                      |---|
| `From`                    | `Help Center <noreply_help.center764705@em872.ramnet.co.za>`                           | Generyczna nazwa, obca domena |
| `Reply-To`                | n/a                                                                                      | n/a |
| `Return-Path`            | `bounces+…@em872.ramnet.co.za`                                                          | Kanał zwrotów z tej samej domeny co MAIL FROM |
| `Received` (ostatni hop) | `wfbttnqp.outbound-mail.sendgrid.net [159.183.114.67]` → serwer odbiorcy               | Relay SendGrid (infrastruktura mailingowa) |
| **SPF**                   | `pass` dla `em872.ramnet.co.za`                                                         | Uwierzytelnia **ramnet.co.za**, nie markę |
| **DKIM**                  | `pass` (d=ramnet.co.za; s=s1)                                                          | Podpis domeny nadawcy technicznego |
| **DMARC**                 | `none`                                                                                  | Brak polityki DMARC (ułatwia nadużycia) |

---

### Tabela wskaźnika IOC

| **Typ**   | **Wartość**                                                                                           | **Kontekst**                                          | **Poziom wiarygodności wskaźnika** |
|---          |---                                                     |---                                            |---|
| **Domena**  | `ramnet.co.za`                                         | Nadawca techniczny (MAIL FROM / DKIM d=)      | Wysoki |
| **Domena**  | `em872.ramnet.co.za`                                   | Subdomena użyta do wysyłki/bounce             | Wysoki |
| **IP**      | `159.183.114.67`                                       | `outbound-mail.sendgrid.net` (relay)          | Średni |
| **URL**     | `https://pos[.]nham24.com/files/files/help.html`         | Główny CTA („UPDATE NOW”)                     | Wysoki |
| **Domena**  | `pos.nham24.com`                                       | Domena docelowa CTA                           | Wysoki |
| **URL**     | (obrazy) `https://cdn.nflximg.com/...` lub proxy       | Branding wizualny wykorzystywany w treści     | Średni |
| **Temat** | `Renew your subscription to avoid any delay on your service` | Presja czasu / utrata usługi            | Średni |
| **Zwrot**  | `Your account is on hold` / `Update your payment method` | Socjotechnika – strach i pośpiech         | Wysoki |
