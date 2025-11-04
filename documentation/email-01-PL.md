# Email 01 - Masowy phishing z narzuceniem presji czasu

Już na etapie wstępnego oglądu — z perspektywy zwykłego odbiorcy — w oczy od razu wpadają dwa elementy: alarmistyczny tytuł „Nie ma już czasu” oraz deklarowany nadawca „UPS”. 
To połączenie samo w sobie uruchamia intuicyjny hamulec: komunikat o skrajnym pośpiechu zwykle nie pasuje do typowego, spokojnego stylu komunikacji firm kurierskich.

- **UPS nie używa komunikatów w stylu „nie ma już czasu”; raczej „Twoja przesyłka czeka na odbiór”, „Twoja przesyłka jest w drodze” itp.**
- **Nie korzystałem z usług kuriera UPS w czasie dostania wiadomości, więc to jest dodatkowa czerwona flaga.**

<p align="center">
  <a href="screenshots/mail1/header1.png">
    <img src="/screenshots/mail1/header2.png" width="600" alt="Widok maila (nagłówek)">
  </a>
</p>
<p align="center">
  <em>Rys. 1 — Nagłówki</em>
</p>

Po wejściu głębiej — podgląd wiadomości w preview w sandboxie — mail od razu przypomina klasyczny phishing. **W głównej mierze dlatego, że jest bardzo słabo wykonany.**
W treści nie widać żadnych elementów potwierdzających, że to faktycznie UPS (brak spójnej domeny nadawcy, brak identyfikatorów przesyłki). 
Pierwszy nagłówek **FROM**, potwierdza klasyczny przypadek podrobienia wyświetlanej nazwy i podszywania się pod znaną firmę **(Brand impersonation)**. 

Szybka inspekcja zawartości i osadzonych linków (po otwarciu wiadomości w formacie .txt) wskazuje na próbę wyłudzenia: **widoczny odnośnik „U.n.s.u.b” prowadzi bezpośrednio na adres IP 5[.]231[.]202[.]248,** a w kodzie umieszczono piksel śledzący z domeny chipcrack[.]net — obie wartości nie mają żadnego związku z domeną UPS. Rozjazd domeny względem marki oraz użycie surowego adresu IP w odnośniku CTA to klasyczne czerwone flagi kampanii masowych.

<div align="center">
  <img src="/screenshots/mail1/message1.png" alt="Podgląd wiadomości" width="35%">
  &nbsp;&nbsp;
  <img src="/screenshots/mail1/screenbody1.png" alt="Widok HTML" width="55%">
</div>
<p align="center"><em>Rys. 2,3 — Podgląd wiadomości + HTML</em></p>

Przy dalszej analizie maila, możemy zaobserować, że to nie jest korespondencja UPS, tylko kampania masowa z własnej infrastruktury atakującego. Pierwsza czerwona flaga, oprócz podrobienia wyświetlanej nazwy, to **Return-Path: <…@chipcrack.es>** — zwrotny adres także wskazuje na chipcrack.es. Ekstremalnie długi „local-part” (ciąg cyfr/liter) wygląda jak znacznik śledzący/identyfikator kampanii (VERP/track ID).

Następnie patrzymy na nagłówek **Received: from aaa[.]altnewlywed[.]shop ([163[.]172[.]189[.]190]).** Tu widzimy, że wiadomość weszła z hosta aaa.altnewlywed.shop na publicznym IP 163[.]172[.]189[.]190. To wygląda jak zwykły VPS serwer z losową domeną, więc to na pewno nie infrastruktura UPS. 

W dalszej analizie przystąpiłem do weryfikacji DKIM i SPF. Podpisy są wystawione dla chipcrack.es (nie ups.com - co jest kolejnym dowodem na phishing) i dodatkowo używają SHA-1 (słaby algorytm i przestarzały). Podpis DKIM dla domeny atakującego nie potwierdza autentyczności rzekomej marki — co najwyżej, że nadawca kontroluje chipcrack.es. **W całym mailu brakuje jednak SPF/DKIM/DMARC od serwera odbiorcy - czyli brakuje sekcji Authentication-Results.**. Jednak jest to też normalne działanie - bramy AV/antyspam czasem stripują lub zamieniają Authentication-Results: na swoje X-*. Mailing listy też potrafią zostawić tylko swoje metadane. Możliwe też, że stare systemy mailngowe w ogóle nie uruchamiają modułów SPF/DKIM/DMARC albo nie loguje wyników w Authentication-Results.

Z innych ciekawych wniosków, możemy zaobserwować, że w środku maila znajduje się link do obrazka na imgur.com. Niestety, po otworzeniu obrazka, nie znajdujemy go nawet na serwerze, co dodatkowo podważa wiarygodność wiadomości. Legalny nadawca (np. UPS) **raczej nie hostuje elementów maila na publicznym serwisie typu imgur.com — używa do tego własnej domeny.** Brak pliku sugeruje „jednorazową” kampanię (rotacja/wyczyszczenie zasobów, usunięcie przez Imgur lub autora), co jest typowe dla masowych phishingów: niechlujnie sklejony szablon, zewnętrzne hosty, krótki czas życia dowodów.
Dalsze wyniki w urlscanio, whois, i mxtoolbox potwierdzają, że była to "masówka" phishingowa, która była wykonana jakiś czas temu.

## Podsumowanie

- **Motyw:** przekierowanie do strony kampanii (prawd. wyłudzenie danych/opłaty - nie można zweryfikować ze względu na wygaszoną kampapanię).
- **Typ ataku:** Prawdopodobnie masowy phishing z podrobieniem marki oraz spoofing wyświetlanej nazwy,
- **Ocena końcowa:** _**Phishing**_

### Analiza URL

##### urlscan.io
**hxxp://chipcrack[.]net/track/.../30285o9**
- jest to niewidoczny piksel śledzący (często przezroczysty) osadzony w treści. Adres tego typu z unikalnym identyfikatorem pozwala nadawcy sprawdzić, czy wiadomość została otwarta i że skrzynka jest aktywna. Dane z piksela służą do mierzenia skuteczności kampanii i segmentowania odbiorców (np. ponowne wysyłki, więcej spamu dla aktywnych skrzynek).

**5[.]231[.]202[.]248**
- Skan zwraca błąd (We could not scan this website). Prawdodobnie link już nie działa prawidłowo, ponieważ prawdopodobnie link już nie działa prawidłowo, ponieważ kampania została wygaszona lub serwer IP przestał odpowiadać / blokuje skanery.

##### VirusTotal 
<div align="center">
  <img src="/screenshots/mail1/virustotal.png" alt="Widok HTML" width="105%">
</div>

##### WHOIS 
**5[.]231[.]202[.]248**
- RIPE region (RIPE NCC). Szczegóły właściciela do pobrania z bazy RIPE (RDAP). Adres nie należy do UPS. Pawdopodobnie serwer pośredni/VPS użyty w kampanii.

##### MXToolbox 
- wedle raportu mozemy wywnioskować że domena ma niską higienę pocztową i nie wygląda na oficjalny kanał wysyłki UPS - brak DMARC, brak/niepoprawny SPF oraz problemy z rekordami MX/DNS.
<div align="center">
  <img src="/screenshots/mail1/mxtoolbox.png" alt="Widok HTML" width="105%">
</div>


### Analiza nagłówków

| Pole                      | Wartość                                                                           | Notatka |
|---                        |---                                                                                  |---|
| `From`                    | `"UPS"` <…@chipcrack[.]es>                                                          | **Spoofing wyświetlanej nazwy** |
| `Reply-To`                | —                                                                                    | n/a |
| `Return-Path`            | <…@chipcrack[.]es>                                                                    | Adres zwrotny wskazuje na domenę nadawcy kampanii. |
| `Received` (ostatni hop) | from aaa.altnewlywed[.]shop ([163.172.189.190])                                       | Ostatni host to VPS (Virtual Private Server) z obcą domeną; brak powiązania z UPS. |
| **SPF**                   | n/a                                                                                  | n/a |
| **DKIM**                  | `d=chipcrack[.]es; a=rsa-sha1; s=smtp`                                               | Podpis dla domeny atakującego, nie UPS |
| **DMARC**                 | n/a                                                                                  | n/a |


### Tabela wskaźnika IOC

| **Typ**   | **Wartość**                                                                                           | **Kontekst**                               | **Poziom wiarygodności wskaźnika** |
|---        |---                                                                                                     |---                                          |---|
| **Domena**| chipcrack[.]es                                                                                         | Envelope-From / Return-Path / DKIM d=       | Wysoki |
| **Domena**| aaa[.]altnewlywed[.]shop                                                                               | Host nadawczy w `Received`                  | Wysoki |
| **IP**    | 163[.]172[.]189[.]190                                                                                  | Źródłowe IP serwera nadawcy                 | Wysoki |
| **Adres URL** | hxxp://5[.]231[.]202[.]248/ShpVWA32333Tpuf133gwzwgzcmey1491.../3028529                             | CTA / „U.n.s.u.b” z maila                   | Wysoki |
| **Domena**| chipcrack[.]net                                                                                        | Tracker / piksel kampanii                   | Średni |
| **Adres URL** | hxxp://chipcrack[.]net/track/3zpkFh32333PFCr133uvidiswzhm1491.../30285o9                          | Tracker / piksel kampanii                   | Średni |
| **Temat** | „nie ma już czasu”                                                                                     | Presja czasu (socjotechnika)                | Średni |
| **Fraza** | „If you no longer wish to receive emails from us, please U.n.s.u.b”                                    | Fraza z kampanii                            | Średni |
