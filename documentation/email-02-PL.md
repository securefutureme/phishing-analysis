# Email 02 - Wyłudzenie danych logowania

### Źródło: www.malware-traffic-analysis.net

Biorąc się do analizy maila pobranego z świetnej strony ze zbiorem malware i maili phishingowych (co jest niesamowitym wsparciem dla każdego, kto chce trenować swoje umiejętności analityczne potrzebne do SOC i nie tylko), widzimy od razu, że mail został skonstruowany tak, żeby przyciągnąć naszą uwagę. Jakbyśmy zamienili się w zwykłego użytkownika, który otwiera skrzynkę mailową i dostaje takiego maila (zakładając, że korzystamy ze strony malware-traffic-analysis.net), dostajemy informację, że mail przyszedł z Supportu znanej nam strony.

Pierwsza czerwona flaga jaka jest tutaj, to rozjechanie się tytułu nadawcy **„malware-traffic-analysis.net Support”,** w stosunku do adresu mailowego nadawcy **sues@nnwifi[.]com**. Z punktu widzenia użytkownika „Support” brzmi wiarygodnie, ale klient pocztowy pokazuje prawdę: nadawcą jest **sues@nnwifi[.]com**, czyli zupełnie inna domena niż ta użyta w tytule nadawcy. 

Dodamy do tego jeszcze **„Warning: Final notice”** - gdzie od razu widać, że to **typowy chwyt socjotechniczny na wywarcie presji czasu**. Widzimy tu dwie duże czerwone flagi, na które zwracamy uwagę przy analizie phishingu.

<p align="center">
  <img src="/screenshots/mail2/header1.png" alt="Podgląd nagłówka" width="60%">
</p>
<p align="center">
  <em>Rys. 1 — Nagłówki</em>
</p>

Zagłębiając się dalej w maila, w treści wiadomości pojawia się prośba o „**potwierdzenie własności adresu**” z linkiem prowadzącym poza domenę odbiorcy. Gdy najedziemy kursorem na link, widzimy **docelową lokalizację**: ``hxxps://servervirto[.]com[.]co/ed/trn/update?email=brad@malware-traffic-analysis.net``. Prawdopodobnie pole Email jest tu wstępnie wypełnione z parametru email= (tracking), czyli zapewnie link przerierowuje do fałszywej strony logowania z wypełnionym mailem, w celu pozyskania haseł.

Adres admin@malware-traffic-analysis.net w stopce jest tutaj zwykłym tekstem (brak mailto:) – tylko wystylizowany na niebiesko.

<p align="center">
  <img src="/screenshots/mail2/body1.png" alt="Podgląd nagłówka" width="60%">
  </p>
  <p align="center">
    <em>Rys. 2 — Treść wiadomości</em>
</p>

Jednakże, jest to jeden ze starszych maili ze strony malware-traffic-analysis.net, obecnie link nie działa - i wyskakuje **błąd 400.** Możemy zakładać, (z uwagi, że to maile które mogą być zmodyfikowane przez mailware-traffic-analysis), że albo nie są to realne linki, albo kampania jest już wygaszona lub hosty blokują skanery. 
Jednak na stronie malware-traffic-net, widzimy załączony screen shoot, co się dzieje po kliknięciu na "Confirm ownership":

<p align="center">
  <img src="/screenshots/mail2/phishing-page.png" alt="Podgląd nagłówka" width="60%">
  </p>
  <p align="center">
    <em>Rys. 3 — Fałszywa strona z logowaniem</em>
</p>

Ewidentnie mamy tutaj przykład **wyłudzania poświadczeń (credential phishing).**

W nagłówkach wiadmości zdobywamy coraz więcej dowodów. Ostatni hop to mail[.]nnwifi[.]com (173[.]46[.]174[.]49) — to z tego serwera wiadomość trafiła do adresata. Niżej widzimy kilka przeskoków po localhost (127.0.0.1) przez amavisd-new (jest to swego rodzaju "pośrednik” między serwerem pocztowym (MTA, np. Postfix) a skanerami treści, który działa na tym samym hoście); to nie są osobne serwery w Internecie, tylko wewnętrzne przekazania w obrębie mail.nnwifi.com. Najważniejszy jest jednak najniższy wpis:

- **Received: from nnwifi[.]com (94[.]100[.]31[.]27) by mail[.]nnwifi[.]com with ESMTP**

Żeby to ubrać w słowa najprościej - nadanie nastąpiło z hosta nnwifi[.]com (IP 94[.]100[.]31[.]27) z użyciem ESMTP (czyli uwierzytelnionego zgłoszenia przez klienta). **Czyli ktoś zalogował się do serwera mail[.]nnwifi[.]com i wysłał wiadomość jako „Support”, po czym lokalny MTA przepuścił to przez swoje filtry i wysłał dalej.** To wygląda na zwykły serwer nadawcy kampanii, **nie** infrastrukturę
malware-traffic-analysis[.]net. Oczywiście, tutaj nie da się mieć 100% pewności bez logów serwera, ale z dostarczonych nagłówków teza o zewnętrznym „legalnym” providerze lub forwardzie jest **mało prawdopodobna**.

**Trzy różne domeny w jednej wiadomości pozwalają nam zatwierdzić wniosek, że jest to phishing. Nadawca (nnwifi.com), marka w nazwie (malware-traffic-analysis[.]net - abstrahując od tego, że to mail sample/zmodyifklwany), oraz cel linku (servervirto[.]com[.]co). Legalne powiadomienia rzadko mieszają tyle przestrzeni nazw.**

---

## Podsumowanie
- **Motyw:** Podszycie się pod wsparcie skrzynki w celu wyłudzenia poświadczeń poprzez fałszywą stronę logowania.
- **Typ ataku:** **Credential phishing** z **podszyciem pod markę** i **display-name spoofing**;
- **Ocena końcowa:** _**Phishing**_

### Analiza URL

##### urlscan.io
**servervirto[.]com**
- Obecnie link nie działa - jak i cała domena servervito.

##### WHOIS 
**173[.]46[.]174[.]49**
Ostatni hop z nazwą mail.nnwifi.com – wygląda na zwykły serwer MTA dostawcy hostingu (geolokacja USA). Nie jest to infrastruktura domeny, pod którą się podszyto.

##### MXToolbox 
**brak DKIM/DMARC/SPF**

### Analiza nagłówków

| Pole                      | Wartość (sanitized)                                                                                               | Notatka |
|---                        |---                                                                                                                |---|
| `From`                    | `"malware-traffic-analysis.net Support"` <sues@nnwifi[.]com>                                                       | **Display name spoofing** |
| `Return-Path`             | n/a (nie dostarczono)                                                                                              | Brak jawnego kanału zwrotnego w wycinku. |
| `Reply-To`                | n/a (nie dostarczono)                                                                                              | Często w phishingu wskazuje inny adres – tu brak danych. |
| `Received` (ostatni hop)  | from **mail.nnwifi[.]com** ([173[.]46[.]174[.]49])                                                                 | Ostatni serwer nadawcy kampanii. |
| **SPF**                   | n/a                                                                                                                | Brak `Authentication-Results` w dostarczonym fragmencie. |
| **DKIM**                  | n/a                                                                                                                | j.w. |
| **DMARC**                 | n/a                                                                                                                | j.w. |


### Tabela wskaźnika IOC

| **Typ**   | **Wartość**                                                                                           | **Kontekst**                               | **Poziom wiarygodności wskaźnika** |
|---      |---                                                                                                      |---                                               |---|
| Domena  | nnwifi[.]com                                                                                            | Domena nadawcy w `From` / serwery `mail.nnwifi` | Wysoki       |
| Domema  | mail.nnwifi[.]com                                                                                       | Ostatni hop w `Received`                        | Wysoki       |
| IP      | 173[.]46[.]174[.]49                                                                                     | IP serwera `mail.nnwifi[.]com`                  | Wysoki       |
| IP      | 94[.]100[.]31[.]27                                                                                      | IP hosta `nnwifi[.]com` (nadanie ESMTP)        | Wysoki       |
| URL     | hxxps://servervirto[.]com[.]co/ed/trn/update?email=brad@malware-traffic-analysis[.]net                  | Link CTA „Confirm ownership”                     | Wysoki       |
| Domema  | servervirto[.]com[.]co                                                                                  | Cel kampanii (credential phishing)              | Wysoki       |
| Temat | `Warning: Final notice`                                                                                 | Socjotechnika – presja czasu                    | Średni     |
| Zdanie  | `Confirm ownership`                                                                                     | Fraza przynęty (weryfikacja konta)              | Średni     |
| Adres email   | sues@nnwifi[.]com                                                                                       | Adres nadawcy niespójny z marką                 | Wysoki       |
