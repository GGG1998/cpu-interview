```
1.2.1.2. Implementacja

Podstawowy mechanizm przerwania działa następująco. Osprzęt procesora ma
ścieżkę nazwaną linią żądania przerwania (interrupt-request line), którą pro-
cesor bada po wykonaniu każdego rozkazu. Jeśli procesor (CPU) wykryje, że
sterownik zasygnalizował coś w linii żądania przerwania, to przechwyci numer
przerwania i wykona skok do procedury obsługi przerwania (interrupt-handler
routine), posługując się numerem przerwania jako indeksem do tablicy wektora
przerwań. Rozpoczyna wtedy wykonywanie rozkazów, poczynając od adresu
skojarzonego z indeksem. Procedura obsługi przerwania przeważnie dotyczy
stanu, który zostanie zmieniony podczas jej działania, wykrywa przyczynę
przerwania, wykonuje niezbędne czynności, a następnie wykonuje rozkaz
powrotu z przerwania (return from interrupt) mający na celu przywrócenie
procesora do stanu wykonywania przerwanego programu. Mówimy, że ste-
rownik urządzenia zgłasza (raises) przerwanie za pomocą sygnału podawanego
w linii żądania przerwania, procesor przechwytuje (catches) przerwanie i eks-
pedjuje je do procedury obsługi przerwania(ISR), a ta czyści przerwanie, obsługując
urządzenie. Na rysunku 1.4 są pokazane elementy cyklu wejścia-wyjścia ste-
rowanego za pomocą przerwania.

Opisany podstawowy mechanizm przerwania umożliwia procesorowi reago-
wanie na zdarzenia asynchroniczne, takie jak wystąpienie w sterowniku
urządzenia gotowości do pracy. W nowoczesnym systemie operacyjnym
potrzebujemy jednak bardziej złożonych możliwości obsługiwania przerwań:

1. Jest nam potrzebne opóźnianie obsługi przerwania wówczas, gdy są wyko-
nywane działania krytyczne.
2. Potrzebujemy sprawnego sposobu przejścia do procedury obsługi przerwa-
nia właściwej dla danego urządzenia.
3. Są nam potrzebne przerwania wielopoziomowe, aby system operacyjny
mógł odróżniać przerwania wysokopriorytetowe od niskopriorytetowych
i reagować z odpowiednim stopniem pilności.

W nowoczesnym sprzęcie komputerowym te trzy własności są zapewniane
przez jednostkę centralną oraz sprzęt sterownika przerwań (interrupt controller
hardware).

Większość procesorów ma dwie linie żądania przerwań. Jedna z nich służy
do przerwań niemaskowalnych (nonmaskable interrupts) i jest zarezerwowana
dla takich zdarzeń jak nieusuwalne błędy pamięci. Druga linia przerwań jest
maskowalna (maskable) — można ją wyłączyć za pomocą procesora przed wykona-
niem krytycznych ciągów rozkazów, których nie wolno przerywać. Przerwania
maskowalne są używane przez sterowniki urządzeń do zgłaszania żądań obsługi.

Przypomnijmy, że celem mechanizmu wektorowego jest zredukowanie ko-
nieczności przeszukiwania przez poszczególne procedury obsługi przerwań
wszystkich możliwych źródeł przerwań, aby rozstrzygnąć, które wymaga ob-
sługi. W praktyce komputery mają więcej urządzeń (a więc i procedur
obsługi przerwań) niż pozycji adresowych w wektorze przerwań. Popularnym
sposobem rozwiązywania tego problemu jest zastosowanie techniki łańcucho-
wania przerwań (interrupt chaining), w której każdy element wektora przerwań
wskazuje na czoło listy procedur obsługi przerwań. Po zgłoszeniu przerwania
procedury obsługowe z odpowiedniej listy są wywoływane jedna po drugiej, aż
zostanie odnaleziona ta, która potrafi obsłużyć dane przerwanie. Mechanizm taki
jest kompromisem między kosztem wielkiej tablicy adresów a nieefektywnością
kierowania przerwania do jednej procedury obsługi.

W skład mechanizmu przerwań wchodzi też system poziomów priorytetów
przerwań (interrupt priority levels). Poziomy te umożliwiają opóźnianie przez
procesor obsługi przerwań niskopriorytetowych bez maskowania wszystkich
przerwań i pozwalają przerwaniom wysokopriorytetowym wyłączać ob-
sługę przerwań niskopriorytetowych.

Podsumowując, przerwania są używane w nowoczesnych systemach opera-
cyjnych do obsługi asynchronicznych zdarzeń (i do innych celów, które
omówimy dalej). Sterowniki urządzeń i błędy sprzętowe powodują zgłaszanie
przerwań. Aby umożliwić wykonywanie większości pilnych prac w pierwszej
kolejności, nowoczesne komputery wykorzystują system priorytetów prze-
rwań. Ponieważ przerwania są używane bardzo intensywnie do przetwarza-
nia, w którym bardzo zależy na czasie, do zapewnienia dobrej wydajności
systemu jest potrzebna sprawna obsługa przerwań.

Na rysunku 1.5 pokazano przykładową tablicę wektorów zdarzeń w proce-
sorach Intela. Zdarzenia z pozycji od 0 do 31 są zarezerwowane; sygnalizują
rozmaite sytuacje takie jak dzielenie przez zero, błąd strony czy brak segmentu.
Zdarzenia o numerach od 32 do 255 są przerwaniami maskowalnymi
generowanymi przez urządzenia.

Numer wektora    Opis
0                 błąd dzielenia
1                 błąd debugowania
2                 wyjątek diagnostyczny
3                 przerwanie puste
4                 punkt kontrolny
5                 nadmiar INTO wykryty
6                 wyjątek granicy przedziału
7                 niedozwolony kod operacji
8                 urządzenie niedostępne
9                 błąd podwojenia
10                przepełnienie segmentu współprocesora (zarezerwowane)
11                niedozwolony segment stanu zadania
12                brak segmentu
13                błąd stosu
14                ochrona ogólna
15                brak strony
16                (zarezerwowane przez firmę Intel — nie używać!)
17                błąd zmiennoprzecinkowy
18                kontrola przylegania
19                kontrola maszyny
19–31             (zarezerwowane przez firmę Intel — nie używać!)
32–255            przerwania maskowalne

Rys. 1.5. Tablica wektorów zdarzeń procesora Intela
```

## Koncept

- **Linie przerwań**: NMI (niemaskowalne) dla krytycznych błędów; IRQ maskowalne do sygnalizacji I/O.
- **Wektor przerwań**: numer → wpis w tablicy wektorów/IDT → adres ISR.
- **Łańcuchowanie przerwań**: wpis wektora wskazuje listę ISR; kolejne ISR sprawdzają, kto obsłuży zdarzenie.
- **Priorytety przerwań**: poziomy umożliwiają obsługę ważniejszych przerwań przed mniej ważnymi bez globalnego maskowania.
- **Kontroler przerwań (PIC/APIC/GIC)**: maskowanie linii, priorytety, kierowanie na rdzenie, EOI (End‑Of‑Interrupt).
- **Prolog/epilog ISR**: zapis niezbędnego kontekstu, minimalna praca w ISR, potwierdzenie przerwania, przywrócenie kontekstu.
- **Dolne połówki**: odroczona praca poza ISR (softirq/tasklet/workqueue) dla dłuższych operacji.

### Przebieg implementacyjny (wysoki poziom)

1. Urządzenie zgłasza IRQ do kontrolera przerwań.
2. Kontroler arbitruje, wyznacza numer wektora i sygnalizuje CPU.
3. CPU kończy bieżący rozkaz, zapisuje kontekst i podnosi poziom priorytetu/maskuje źródło.
4. CPU pobiera adres z tablicy wektorów (IDT) i skacze do ISR.
5. ISR: odczyt statusu urządzenia, przeniesienie/oznaczenie danych, wysłanie EOI do kontrolera.
6. ISR zleca dalszą pracę do dolnej połówki i wraca (instrukcja „return from interrupt”).
7. CPU przywraca kontekst i wznawia przerwany kod użytkownika/jądra.

