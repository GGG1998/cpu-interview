# 1.2.1. Przerwania

```
Rozważmy typowe działanie komputera — program wykonujący operacje
wejścia-wyjścia. Aby rozpocząć operację we-wy, moduł obsługi urządzenia
umieszcza zawartość odpowiednich rejestrów w sterowniku urządzenia. Wte-
dy sterownik urządzenia sprawdza zawartość tych rejestrów, żeby określić,
jaką czynność ma podjąć (np. „czytaj znak z klawiatury”). Sterownik zaczyna
przesyłanie danych z urządzenia do swojego lokalnego bufora. Gdy przysyła-
nie danych się zakończy, sterownik urządzenia poinformuje moduł obsługi
urządzenia, że zakończył działanie. Moduł obsługi przekazuje wówczas ste-
rowanie do innych części systemu operacyjnego, być może zwracając dane
lub wskaźnik do danych, jeśli była to operacja czytania. W przypadku nieu-
danej operacji moduł obsługi zwraca informacje o stanie, na przykład „pisanie za-
kończone pomyślnie” lub „urządzenie zajęte”. W jaki sposób jednak (sprzęto-
wy) sterownik informuje (programowy) moduł obsługi, że zakończył operację?
Robi to za pomocą przerwania (interrupt).

1.2.1.1. Przegląd

Sprzęt może spowodować przerwanie w dowolnej chwili, wysyłając sygnał do
procesora, zwykle za pośrednictwem szyny systemowej. W systemie kompu-
terowym może być wiele szyn, lecz szyna systemowa służy do komunikacji
między jej głównymi komponentami.

Procesor (jednostka centralna) po otrzymaniu sygnału przerwania wstrzy-
muje aktualnie wykonywaną pracę i natychmiast przechodzi do ustalonego
miejsca w pamięci. Miejsce to zawiera na ogół adres startowy procedury ob-
sługującej dane przerwanie. Następuje wykonanie procedury obsługi przerwa-
nia, po zakończeniu której jednostka centralna wznawia przerwane obliczenia.
Przebieg czasowy tych operacji widać na rysunku 1.3.

Przerwania są ważnym elementem architektury komputera. Poszczególne
rodzaje przerwań mogą wymagać odmiennego postępowania. Przerwanie musi
przekazywać sterowanie do odpowiedniej procedury obsługującej przerwanie.
Prostym sposobem rozwiązania tego problemu byłoby wywołanie procedury
ogólnej sprawdzającej rodzaj przerwania i przekazującej sterowanie do właś-
ciwej procedury, ale ta metoda jest powolna, bo wymaga kilku kroków obsłu-
gi przerwania. Jedna z powszechnie stosowanych technik przewiduje, że wyko-
rzystuje się bardziej bezpośrednie podejście. Procedura obsługi przerwania jest
więc wywoływana wprost przez sprzęt.

Ta tablica wskaźników (do procedur obsługujących przerwania) jest zwykle
przechowywana w pamięci dolnej, zajmując w niej — powiedzmy — pierw-
szych sto komórek. Nosi ona nazwę wektora przerwań (interrupt vector) i jest
adresowana jednoznacznie numerem urządzenia podanym w zamówieniu
(żądaniu) przerwania, dzięki czemu otrzymuje się właściwy adres procedu-
ry obsługującej przerwanie zgłoszone przez dane urządzenie. Nawet tak róż-
ne systemy operacyjne jak Windows i UNIX kierują przerwania do obsługi
w opisany sposób.

W architekturze przerwań trzeba również uwzględniać przechowywanie
informacji o stanie obsługi, aby można było go odtworzyć po
obsłużeniu przerwania. Jeżeli procedura obsługi chce zmienić stan procesora — na
przykład przez zmodyfikowanie wartości rejestrów — to musi jawnie przechować
stan bieżący, a przy końcu swojego działania musi go odtworzyć. Po obsłużeniu
przerwania następuje pobranie do licznika rozkazów przechowanego adresu
powrotnego i wznowienie przerwanych obliczeń, tak jakby przerwania nie było.

3 Dalej będziemy używać krótszych określeń: moduł sterujący lub moduł obsługi.
W obiegu jest również nazwa „sterownik”, a nawet „drajwer”; takie jej przypisanie
koliduje z nazwą sterownika w znaczeniu sprzętowym (trzeba by go wówczas określić
mianem „kontrolera”). Jeszcze inna spotykana nazwa to program sterujący urządzenia
— przyp. tłum.


```
**Rules**
- Nasz OS musi mieć moduł obsługi sterownika: [[organizacja_komp#^device-driver-interfejs]] 
- Sprzęt(myszka, klawiatura, monitor, sieć etc.) może spowodować przerwanie w dowolnej chwili, wysyłając sygnał do
procesora, zwykle za pośrednictwem **szyny systemowej**(Architektura Von Neumana)

## Koncept
- **Przerwanie (interrupt)**: asynchroniczny sygnał od urządzenia, żądający uwagi CPU.
- **Pułapka (trap)**: synchroniczne „przerwanie” z CPU (np. wywołanie systemowe, błąd dzielenia).
- **Procedura obsługi przerwania (ISR)**: krótki kod w jądrze; zapis/odtworzenie kontekstu, odczyt statusu urządzenia, potwierdzenie (ACK), minimalna praca.
- **Kontroler przerwań (PIC/APIC/GIC)**: maskowanie, priorytety, kolejkowanie i kierowanie przerwań do wybranych CPU.
- **Sterownik/driver**: ustawia rejestry kontrolera urządzenia; po zakończeniu operacji dostaje przerwanie i kończy I/O.
- **Bufor urządzenia i DMA**: dane trafiają do lokalnego bufora kontrolera lub bezpośrednio do pamięci przez DMA (mniej pracy CPU).
- **Przerwania zegarowe**: tyk systemowy do planisty, odmierzania czasu, time‑outów.
- **Sekcje krytyczne**: czasowe maskowanie przerwań/spinlocki chronią dane; wpływ na latencję.
- **Architektura Von Neumana** - https://lo1.lebork.pl/doc/budowa_komputera_v1.pdf

### Typowy przebieg operacji I/O
1. Driver zapisuje rejestry kontrolera (co, skąd/dokąd, ile).
2. Urządzenie rozpoczyna transfer do swojego bufora lub do RAM (DMA).
3. Po zakończeniu generuje przerwanie do kontrolera przerwań.
4. CPU zapisuje kontekst i skacze do ISR; ISR odczytuje status, potwierdza, przenosi niewielką porcję danych/zdarzeń do kolejek.
5. Dalsza, cięższa praca jest odroczona do „dolnej połowy” (softirq/workqueue) i wykonywana poza ISR.

### TODO
"Procedura obsługi przerwania jest więc wywoływana wprost przez sprzęt. " - Co to znaczy?