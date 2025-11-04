# Email 03 - Masowa kampania reklamowa

<div align="center">
  <img src="/screenshots/mail3/message1.png" alt="Widok HTML" width="65%">
</div>
<p align="center"><em>Rys. 1 — Wiadomość</em></p>

Tak jak w poprzednich przykładach, przy analizie mailów phishingowych, na samym początku zwracamy uwagę na najbardziej rzucające się w oczy "rozwarstwienia", zwykle jest to odbieganie tematu od treści wiadomościu. Jako, że to jest email ze skrzynki spam, nie spodziewałem się tutaj na wykonanie zaawansowanej techniki phishingowej. Mamy oczywiście tutaj pierwszą czerwoną flagę, **czyli temat odbiegający od treści wiadomości - w temacie mamy powiadomienie kurierskie UPS, a w środku mamy agresyjną ofertę „Kup 2 produkty, a zapłacisz tylko za 1”.** Jest to oczywiście To klasyczny przykład phishingu, gdzie nagłówek podszywa się pod komunikat logistyczny, ale treść to schemat sprzedażowy lub wstęp do phishingu.

Im dłużej przeglądamy maila, tym więcej sygnałów, że ktoś próbuje nas ooszukać.  **Kolejny dowód to sam nadawca: `„newsletter-NVH@xykTXwwtlvHiDsl.com”` który pisze w imieniu „Drmerritz”.** 
Poważne firmy nie używają losowych ciągów znaków w adresach ani domen — mają spójne aliasy (np. `no-reply@google.com` `news@updates.ubisoft.com`) i podpisują maile tą samą domeną. 
Przypomina to maile używane do kampanii reklamowych użytych z losowo nazwanego newslettera, który rzekomo pisze w imieniu marki. 

Inną typową techniką — tutaj bardziej chwytem reklamowym niż stricte phishingiem — jest temat wiadmości w stylu „Nie przegap okazji!”. Zarówno w phishingu, jak i w marketingu, nadawcy grają na "FOMO" (strachu przed utratą korzyści). To często pierwsze, co wpada użytkownikowi w oko i bywa, że ciekawość wygrywa z ostrożnością, nawet podprogowo. W praktyce takie komunikaty skraca­ją czas na refleksję i zwiększają szansę na bezrefleksyjne kliknięcie, zwłaszcza gdy towarzyszy im licznik, „limit miejsc” lub duży, kontrastowy przycisk CTA (wezwanie do działania).

<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/message2.png" width="600" alt="Widok maila (nagłówek)">
  </a>
</p>
<p align="center">
  <em>Rys. 2 — Link z wiadomości</em>
</p>

Analizując same nagłówki wiadomości, przyglądamy się łańcuchowi Received:

- **from `swpi.edu.cn` (HELO `mta4.email.shopify.com`) `[167.172.61.220]`  do serwera adresata.**

wiadomość przyszła z hosta identyfikującego się jako `swp[.edu.cn` (domena chińskiej uczelni), ale w komendzie HELO/EHLO przedstawiła się jako `mta4.email.shopify.com`.
Za dużo tutaj niespójności - host wskazuje instytucję (?) akademicką, natomiast **HELO** (jest to komenda protokołu SMTP, którą serwer wysyłający podaje na początku rozmowy z serwerem odbiorcy, aby przedstawić swoją nazwę hosta) podszywa się pod infrastrukturę komercyjnej platformy e-commerce. Taki rozjazd jest nienaturalny dla legalnych wysyłek (gdzie PTR/HELO/SPF zwykle są spójne) i sugeruje nieprawidłową konfigurację lub celowe maskowanie pochodzenia.

<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/message2.png" width="600" alt="Widok maila (nagłówek)">
  </a>
</p>
<p align="center">
  <em>Rys. 3 — Nagłówki</em>
</p>

Dodatkowo w nagłówkach możemy znaleźć:

**- W polu nadawcy pojawia się adres w stylu `…@IjzSRhesiGzCDRg.pl.com.`** Takie "zlepki" często należą do masowo rejestrowanych, niskiej jakości domen.

**- X-XX-DKIM-Status: no signature** – brak podpisu; czyli wiadomość nie ma kryptograficznego podpisu, który potwierdzałby, że naprawdę wysłała ją dana domena i nikt jej nie zmienił po drodze.

Bazując na kolejnym dowodzie, widzimy, że najeżdzając kursorem na przycisk **"Skorzystaj z okazji teraz"**, nie mamy nawet przekierowania do linku. Przycisk bowiem jest "pusty", nie ma odnośnika do realnej strony. Ciekawe zagranie, bowiem może to zwykłego użytkownika zdenerwować. A co robi użytkownik, który widzi bezużytecznego maila? Chce się... wypisać. No i tutaj mamy kolejne ciekawe zagranie, bowiem jedyny aktywny link z maila mamy właśnie w "Wypisz się z newstellera". 

Ten link prowadzi nie do strony rezygnacji z newstellera, tylko w łanczuchu przekierowań (przez system trackingowy) - **`hxxps://dts.innovid.com/.../vclk?…&click=https://cdn.lasvegasusa.eu/...`**
do strony docelowej - **strona kasyna `cdn.lasvegasusa.eu`** W praktyce:

- zamiast się wypisać — potwierdzamy, że skrzynka jest aktywna,
- trafiasz na stronę komercyjną/niezaufaną, niezwiązaną z rzekomym nadawcą,
- zostawiasz ślad w analityce kampanii, co napędza kolejne wysyłki.

## Podsumowanie

- **Motyw:** przekierowanie do strony kampanii (prawd. wyłudzenie danych/opłaty - nie można zweryfikować ze względu na wygaszoną kampapanię).
- **Typ ataku:** masowa kampania phishingowa albo scam z podszywaniem się pod markę (UPS), fałszowaniem nazwy nadawcy/tematu oraz łańcuchem przekierowań opartym na linkach.
- **Ocena końcowa:** _**Scam połączony z phishingiem**_

### Analiza URL

##### urlscan.io

`cdn[.]lasvegasusa[.]eu` 
- jedną z informacji, które możemy tutaj wyciągnąć to, że "cdn." może służyć jako CNAME do dostawcy (np. Cloudflare, Akamai, CloudFront). Na orginalnej stronie **lasvegasusa** sprawdziłem poprzez DevTools -> Network czy są one używane, i faktycznie były. Jednak samo otwarcie `cdn.lasvegas` (poprzez urlscanio) obecnie prowadzi donikąd. Można zakładać, że sam redirect obecnie nie działa, albo kampania reklamowa jest zmieniona/wygaszona.

<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/urlscanio.png" width="600" alt="Widok maila (nagłówek)">
  </a>
</p>

**`setblive.net`** oraz **`swpi.edu.cn`** 
- obecnie nie prowadzą do żadnej strony. Te strony oczywiście mogłybyć zostać zmyślone, albo też mogą tylko przypominać prawdziwe strony - przykładowo `swpi.edu.cn` nie istnieje ale `sxpi.edu.cn` (inny przykład to `swip.ac.cn`) już tak:

<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/typosquatting.png" width="600" alt="Widok maila (nagłówek)">
  </a>
</p>

##### WHOIS 
`167.172.61.220` - z łańcucha Received:
Według WHOIS adres należy do puli `167.172.0.0/16`, przypisanej w regionie RIPE. Ta pula jest kojarzona z infrastrukturą **DigitalOcean (AS14061)**, czyli chmurą/VPS-ami, a nie z Shopify. Jeżeli wiadomość deklaruje się jako `mta4.email.shopify.com`, ale realny IP to `167.172.61.220` (DigitalOcean), mamy zatem niespójność, typową przy podszywaniu pod markę (HELO/EHLO można łatwo sfałszować).


##### MXToolbox 

MXToolbox dla domeny `swpi.edu.cn` pozwala nam umocnić poprzedni wniosek - swpi.edu nie ma rekordów pocztowych, zespoofowane HELO/ENHLO z adresu `167.172.61.220`
<p align="center">
  <a href="screenshots/mail2/header1.png">
    <img src="/screenshots/mail3/mxtoolbox.png" width="600" alt="Widok maila (nagłówek)">
  </a>
</p>

### Analiza nagłówków

| Pole                      | Wartość                                                                                                 | Notatka |
|---                        |---                                                                                                      |---|
| `From`                    | "Drmerritz" `<VHmYXKCAMiyxbbz@IKVUybJqdSUIJkm.com>`                                                     | Wyświetlana marka nie jest domeną nadawcy |
| `Reply-To`                | `<newsletter-TSF@PkNLTsZYHCitHIJ.com>`                                                                    | Inna losowa domena niż `From`/brand |
| `Return-Path`             | `<TfGxnDUaSOePORZ@IjzSRhesiGzCDRg.com>`                                                                 | Często realny kanał zwrotu; kolejna losowa domena |
| `Received` (ostatni hop)  | from **`swpi.edu.cn`** (HELO **`mta4.email.shopify.com`**) `[167.172.61.220]`                           | Rozjazd PTR/HELO (uczelnia vs. Shopify) oraz IP niepowiązane z marką |
| **SPF**                   | n/a                                                                                                     | n/a |
| **DKIM**                  | `X-XX-DKIM-Status: no signature (id: n/a)`                                                              | Brak podpisu DKIM |
| **DMARC**                 | n/a                                                                                                     | n/a |



### Tabela wskaźnika IOC

| **Typ**   | **Wartość**                                                                                           | **Kontekst**                                          | **Poziom wiarygodności wskaźnika** |
|---         |---                                                                                                 |---                                                     |---|
| **Domena** | `IKVUybJqdSUIJkm.com`                                                                               | Domena z pola `From` (losowa/niebrandowa)               | Wysoki |
| **Domena** | `IjzSRhesiGzCDRg.com`                                                                          | Domena z `Return-Path` (kanał zwrotu)                        | Wysoki |
| **IP**     | `167.172.61.220`                                                                               | Adres źródłowy w `Received` (`swpi.edu.cn` / HELO Shopify)     | Średni |
| **URL**    | `https://dts.innovid.com/clktru/action/vclk?...&click=https://cdn.lasvegasusa.eu:443/...`       | Przekierowanie/tracker -> landing kasyno                   | Wysoki |
| **Domena** | `cdn.lasvegasusa.eu`                                                                           | Końcowy landing (kasyno, niezwiązany z „UPS”)               | Średni |
| **URL**    | `https://setblive.net/ZyjW455574401la14455EJOcukS `                                                 | „Unsubscribe” prowadzący przez zewnętrzny serwis     | Wysoki |
| **Temat**| 📦 Aktualizacja śledzenia UPS: [...]                                                                    | Marka/temat nie zgadza się z domeną nadawcy      | Wysoki |
| **Zwrot** | „Nie przegap okazji!”                                                                                   | Presja marketingowa w treści                     | Niski |


