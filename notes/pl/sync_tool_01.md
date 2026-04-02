### Przypomnienie
[przerwania](./przerwania.md) oraz [przerwania](./przerwania_implementacja.md)
[IMG_8893.png](../../png/IMG_8893.png) - odczytywanie rozkazów z pamięci
[IMG_8895.png](../../png/IMG_8895.png) - sekcje procesu, licznik rozkazów, zawartość rejestru procesora
[IMG_8898.png](../../png/IMG_8898.png) - blok kontrolnu(stan procesu, rejestry CPU)
[IMG_8904.png](../../png/IMG_8904.png) - context switching, reprezentacja kontekstu odbywa się w bloku kontrolnym

# Case study
Model: producer-consumer
Cel: Przedstawić klasyczny problem producenta-konsumenta z buforem cyklicznym
     we współbieżnym programie

Producent:
    - Wytwarza dane (elementy) i wkłada je do wspólnego bufora
    - Jeśli bufor jest pełny (licznik == ROZMIAR_BUFORA) — czeka   

Konsument:
    - Pobiera dane z bufora i je "konsumuje" (przetwarza)
    - Jeśli bufor jest pusty (licznik == 0) — czeka 

Bufor cykliczny:
    - Tablica o stałym rozmiarze, w której wskaźniki we (wejście) i wy (wyjście) 
      "zawijają się" na początek po dojściu do końca (% ROZMIAR_BUFORA)

### Problem wyścigu (race condition)

Operacje `licznik++` i `licznik--` nie są atomowe — składają się z 3 instrukcji maszynowych:

Niech rejestr eax producenta = będzie rejestr1
Niech rejestr eax konsumera = będzie rejestr2
Niech rejestr1 i rejestr2 są tym samym rejestrem fizycznym

"zawartość takiego rejestru może podlegać przechowywaniu i odtwarzaniu przez ISR"

A to znaczy, że procedura ta zapisuje stan do bloku kontrolnego i go odtwarza, czyli będzie nadpisywać
tymsamym NIE BĘDĄ się synchronizować

```asm
; licznik++ (producent)
mov  eax, [licznik]     ; odczytaj z pamięci do rejestru
add  eax, 1             ; dodaj 1
mov  [licznik], eax     ; zapisz z powrotem

; licznik-- (konsument)
mov  eax, [licznik]     ; odczytaj z pamięci do rejestru
sub  eax, 1             ; odejmij 1
mov  [licznik], eax     ; zapisz z powrotem
```

### Przykład przeplatania (licznik = 5):

```
Producent: mov eax, [licznik]   ; eax = 5
Konsument: mov eax, [licznik]   ; eax = 5  (jeszcze stare!)
Producent: add eax, 1           ; eax = 6
Konsument: sub eax, 1           ; eax = 4
Producent: mov [licznik], eax   ; licznik = 6
Konsument: mov [licznik], eax   ; licznik = 4  ← powinno być 5!
```

Obie operacje się wykonały, ale wynik to 4 zamiast 5 — jedno dodanie "zniknęło" przez brak synchronizacji.

## Kod z książki (C++)

```cpp
#include <iostream>
#include <thread>

const int ROZMIAR_BUFORA = 5;

int bufor[ROZMIAR_BUFORA];
int we = 0;
int wy = 0;
int licznik = 0;

void producent() {
    while (true) {
        int nast_prod = rand(); // produkuj jednostkę

        while (licznik == ROZMIAR_BUFORA)
            ; // nic nie rób

        bufor[we] = nast_prod;
        we = (we + 1) % ROZMIAR_BUFORA;
        licznik++;
    }
}

void konsument() {
    while (true) {
        while (licznik == 0)
            ; // nic nie rób

        int nast_kons = bufor[wy];
        wy = (wy + 1) % ROZMIAR_BUFORA;
        licznik--;

        // konsumuj jednostkę z nast_kons
        std::cout << nast_kons << std::endl;
    }
}

int main() {
    std::thread t1(producent);
    std::thread t2(konsument);
    t1.join();
    t2.join();
}
```

# Sekcja krytyczna
**Sekcja krytyczna** - segment kodu, który jest współdzielony z innym procesem
**Problem sekcji krytycznej** - jak zaprojektować kooperacje pomiędzy procesami chcący synchronizować swoją aktywność
**Kryteria rozwiązania problemu sekcji krytycznej**
    - Wzajemne wykluczenie: Jeśli proces P działa w swojej sekcji krytycznej, żaden inny proces nie działa w sekcji krytycznej
    - Postęp: 
    - Ograniczone czekanie

## Obsługa sekcji krytycznej w OS:
- Jądro wywłaszczające: wywłaszczenie procesu w trybie jądra
- Jądro niewywłaszczające: proces będzie wykonywany do wyjścia z trybu jądra, albo dopóki się nie zablokuje
  lub dobrowolnie nie odda kontroli nad CPU

## Rozwiazanie Petersona

> UWAGA: to tylko klasyczna prezentacja rozwiazania. Na wspolczesnych architekturach moze nie dzialac
> poprawnie ze wzgledu na reordering instrukcji `load`/`store`. W praktyce uzywa sie operacji atomowych
> lub barier pamieci.

Rozwiazanie **programowe** (software-based) dla **2 procesow** (P_i i P_j, gdzie j = 1 - i).

### Wspoldzielone dane

```c
int ruch;              // czyja kolej na wejscie do sekcji krytycznej
boolean znacznik[2];   // czy dany proces chce wejsc
```

- `ruch` - wskazuje ktoremu procesowi przypada kolej
- `znacznik[i] == true` - proces P_i jest gotowy do wejscia do sekcji krytycznej

### Struktura procesu P_i

```c
while (true) {
    znacznik[i] = true;       // chce wejsc. LOCK()
    ruch = j;                 // ustepuje drugiemu
    while (znacznik[j] && ruch == j)
        ;                     // czekaj (busy wait)

    /* sekcja krytyczna */

    znacznik[i] = false;      // wychodzi. UNLOCK()

    /* reszta */
}
```

Proces ustawia swoj znacznik i oddaje pierwszeństwo drugiemu. Wchodzi do SK gdy:
- drugi proces nie chce wejsc (`znacznik[j] == false`), **lub**
- jest jego kolej (`ruch == i`)

### Dowod poprawnosci

1. **Wzajemne wykluczanie** - P_i wchodzi tylko gdy `znacznik[j] == false` lub `ruch == i`.
   `ruch` moze byc 0 albo 1 - nie obydwoma naraz. Wiec oba procesy nie moga jednoczesnie przejsc przez `while`.

2. **Postep** - P_i moze utknac w petli `while` tylko gdy `znacznik[j] == true && ruch == j`.
   Jesli P_j nie chce wejsc -> `znacznik[j] == false` -> P_i natychmiast przechodzi.

3. **Ograniczone czekanie** - jesli P_j wychodzi z SK, ustawia `znacznik[j] = false` -> P_i wchodzi.
   Jesli P_j chce wejsc ponownie, ustawia `ruch = i` -> P_i wchodzi. Czekanie co najwyzej jedno.

### Dlaczego nie dziala na wspolczesnych architekturach

Procesory i kompilatory moga zmieniac kolejnosc operacji load/store dla wydajnosci.

**Przyklad:**

```c
// wspoldzielone dane
boolean znacznik = false;
int x = 0;

// watek 1:              // watek 2:
while (!znacznik)        x = 100;
    ;                    znacznik = true;
print x;
```

Oczekujemy `x = 100`, ale procesor moze przestawic instrukcje watku 2
(`znacznik = true` PRZED `x = 100`) -> watek 1 przeczyta `x = 0`.

**Wplyw na Petersona** - jesli procesor przestawi kolejnosc w sekcji wejsciowej:

```
Proces_0: -> ruch = 1 ──────────────────> znacznik[0] = true ──> SK
Proces_1: ────> ruch = 0, znacznik[1] = true ──────> SK

Czas ──>
```

Oba procesy trafiaja do sekcji krytycznej jednoczesnie - **wzajemne wykluczanie zlamane**.

**Wniosek:** jedynym sposobem zapewnienia wzajemnego wykluczania sa sprzetowe narzedzia synchronizacji
(bariery pamieci, instrukcje atomowe).

