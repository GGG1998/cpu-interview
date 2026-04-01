# 1.2 Organizacja systemu komputerowego — kluczowe koncepcje

## Oryginał
```
1.2. Organizacja systemu komputerowego

Współczesny system komputerowy ogólnego przeznaczenia składa się z jednego lub więcej procesorów2 i pewnej liczby (sprzętowych) sterowników urządzeń (device controllers) połączonych wspólną szyną (magistralą — bus) umożliwiającą kontakt ze wspólną pamięcią (rys. 1-2). Każdy sterownik urządzenia odpowiada za określony typ urządzenia (np. za napędy dysków, urządzenia dźwiękowe lub wyświetlacze określonego typu). Do sterownika, zależnie od jego typu, można podłączyć więcej niż jedno urządzenie. Na przykład jeden systemowy port USB można podłączyć do koncentratora („huba”) USB, do którego można podłączyć kilka urządzeń. Sterownik urządzenia zarządza pewną ilością lokalnej pamięci buforowej i zbiorem wyspecjalizowanych rejestrów. Sterownik taki odpowiada za przemieszczanie danych między urządzeniami zewnętrznymi, nad którymi sprawuje nadzór, a wspólną lokalną pamięcią buforową.

Dla każdego sterownika urządzenia systemy operacyjne mają zwykle
moduł sterujący urządzenia (device driver3). Ów moduł sterujący „rozumie”
sterownik (rozpoznaje jego sygnały) i tworzy dla reszty systemu operacyjnego
jednolity interfejs do urządzenia. Jednostka centralna i sterowniki urządzeń
mogą pracować równolegle, rywalizując o cykle pamięci. Zapewnienie upo-
rządkowanego dostępu do pamięci jest zadaniem sterownika pamięci, który
ten dostęp synchronizuje. ^device-driver-interface

Zaczynamy od przerwań, które powiadamiają jednostkę
centralną o zdarzeniach wymagających uwagi. Potem omawiamy strukturę
pamięci i budowę wejścia-wyjścia.
```

## Koncept
- **Magistrala (bus) i wspólna pamięć**: centralny kanał komunikacji łączący procesory, sterowniki urządzeń i pamięć.
zestaw przewodów łączących komponenty
Fizycznie to po prostu ścieżki elektryczne na płycie głównej. Są dwa główne typy:
Szyna adresowa (address bus)

"GDZIE" — CPU podaje adres komórki pamięci, do której chce się dostać.

- Jednokierunkowa: CPU → pamięć
- Szerokość determinuje ile pamięci można zaadresować
- 16 bitów → 2^16 = 64 KB
- 32 bity → 2^32 = 4 GB
- 64 bity → teoretycznie 16 EB (exabajtów)

Szyna danych (data bus)

"CO" — tędy przepływają faktyczne dane (wartości, rozkazy).

- Dwukierunkowa: CPU ↔ pamięć
- Szerokość określa ile danych na raz można przesłać
- 8 bitów → 1 bajt na raz
- 32 bity → 4 bajty na raz
- 64 bity → 8 bajtów na raz

Jak to działa razem

Odczyt z pamięci:
1. CPU wystawia adres na szynę adresową:  [0x00FF]  →  pamięć
2. Pamięć odsyła dane szyną danych:      CPU  ←  [01100001]

Zapis do pamięci:
1. CPU wystawia adres na szynę adresową:  [0x00FF]  →  pamięć
2. CPU wysyła dane szyną danych:          CPU  →  [01100001]  →  pamięć

Jest też trzecia: szyna sterująca (control bus)

"JAK" — sygnały typu: odczyt/zapis, przerwanie, zegar. Koordynuje całą komunikację.

- **Wiele procesorów (CPU)**: system może mieć więcej niż jeden procesor; implikacje dla współdzielenia zasobów.
- **Sterowniki urządzeń (device controllers)**: sprzętowe kontrolery przypisane do typów urządzeń (dyski, audio, wyświetlacze itp.).
- **Lokalna pamięć buforowa kontrolera**: buforowanie danych na poziomie kontrolera przed/po transferze do wspólnej pamięci.
- **Rejestry specjalizowane**: konfiguracja i status pracy kontrolera; interfejs programowy do I/O.
- **Topologia urządzeń**: jeden kontroler może obsługiwać wiele urządzeń (np. hub USB pod jednym portem).
- **Przemieszczanie danych**: rola kontrolera w transferze między urządzeniami zewnętrznymi a wspólną pamięcią.
- **Terminologia**: „sterownik urządzenia” jako kontroler sprzętowy vs. „driver” w systemie operacyjnym.
- **(Dalej w 1.2.1) Przerwania**: mechanizm powiadamiania CPU o zdarzeniach I/O i zakończeniach operacji.

### Dalsze koncepcje (driver, równoległość, arbitraż pamięci)
- **Device driver (programowy moduł OS)**: rozumie protokół kontrolera i wystawia jednolity interfejs do reszty systemu.
- **Abstrakcja urządzeń**: ujednolicenie obsługi różnych kontrolerów pod wspólnym API OS.
- **Równoległość CPU i kontrolerów**: wykonywanie niezależne; współzawodnictwo o cykle pamięci.
- **Sterownik pamięci / kontroler pamięci**: synchronizuje i porządkuje dostęp wielu uczestników do wspólnej pamięci (arbitraż).
- **Ścieżka do dalszej nauki**: przerwania → hierarchia/priorytety; organizacja pamięci → spójność/koherecja; wejście–wyjście → tryby PIO/DMA.

## Przemyślenia ???
- Mikrokontrolery(MCU) to taki układ, który ma CPU, RAM, każdy takie CPU ma swój protokuł, czyli w jaki sposób będzie odbierać sygnał i go przetwarzać. 
- Takie mikrokontrolery mają swoją też pamieć, więc mogą przyjmować rozkazy, które definiują co będzie na wyjściu
- UWAGA! Rozkazy są częścią procesora(różne architektury, różne kodowanie), historia tego jest prosta:
    - Każdy rozkaz to liczba 10110000 01100001 zakodowana w pamięci
    - TO KOSZMAR, więc wprowadzają mnemoniki takie jak np: `MOV AL, 0x61    →    10110000 01100001`

