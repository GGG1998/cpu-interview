```
flowchart LR
 subgraph CPU["Procesor (CPU)"]
        A["Moduł sterujący urządzenia<br>rozpoczyna operację wejścia-wyjścia"]
        B["Między rozkazami procesor<br>sprawdza występowanie przerwań"]
        C["Procesor otrzymuje przerwanie i przekazuje sterowanie<br>procedurze obsługi przerwania"]
        D["Procedura obsługi przerwania przetwarza dane<br>i wykonuje powrót po przerwaniu"]
        E["Procesor podejmuje wykonywanie<br>przerwanego zadania"]
  end
 subgraph IO["Sterownik wejścia-wyjścia"]
        F["Rozpoczęcie operacji wejścia-wyjścia"]
        G["Gotowość wejścia, zakończenie wyjścia<br>lub błąd powoduje wytworzenie sygnału przerwania"]
  end
    A -- 2 --> F
    F -- 3 --> G
    G -- 4 --> C
    C -- 5 --> D
    D -- 6 --> E
    E -- 7 --> B
    B -- przerwanie? --> C
```