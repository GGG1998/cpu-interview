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