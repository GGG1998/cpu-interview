# Przypomnienie
[Prąd](./elek_00.md)
[Sygnał](./elek_sygnalyIO.md)
[Rezonator](./elek_rezonator.md)
[Oscylator, generator](./elek_oscylator.md)

# Zegar CPU - jak dziala i jak go zbudowac

## Zrodlo sygnalu zegarowego

Na najnizszym poziomie potrzebujesz oscylatora kwarcowego. Kwarc piezoelektryczny wibruje mechanicznie z bardzo stabilna czestotliwoscia (np. 25 MHz). Typowy obieg:

```
Krysztal kwarcowy -> Oscylator -> PLL (mnoznik czestotliwosci) -> Dystrybucja zegara
```

PLL (Phase-Locked Loop) mnozy bazowa czestotliwosc. Np. krysztal 25 MHz x 160 = 4 GHz. Wiekszosc FPGA i CPU ma wbudowane bloki PLL.

## Na poziomie elektrycznym

Zegar to po prostu sygnal napieciowy przelaczajacy sie miedzy stanem niskim (0V) a wysokim (np. 3.3V lub 1.2V):

```
     ┌───┐   ┌───┐   ┌───┐
     │   │   │   │   │   │
─────┘   └───┘   └───┘   └───
     <-T->
     T = okres (1/czestotliwosc)
```

## Co robi zegar w CPU

### Przerzutnik (flip-flop)

Przerzutnik to najprostszy element pamieci w cyfrówce - przechowuje jeden bit (0 lub 1).

- Ma wejscie danych (`D`) i wejscie zegara (`CLK`)
- Przez caly czas "patrzy" na wejscie D, ale **ignoruje je**
- Dopiero w momencie zbocza narastajacego zegara (przejscie 0->1) kopiuje wartosc z D na swoje wyjscie Q
- I **trzyma** te wartosc az do nastepnego zbocza

```
         ┌─────────┐
  D ────>│         │
         │ Flip-   ├────> Q (wyjscie - zatrzasnieta wartosc)
 CLK ───>│  Flop   │
         └─────────┘
```

**"Zatrzaskiwanie"** to wlasnie ten moment - flip-flop "lapie" dane z wejscia i je zamraza.

### Dlaczego to jest potrzebne?

Bramki logiczne (AND, OR, NOT) polaczone w lancuch nie przenosza sygnalu natychmiast - kazda bramka dodaje male opoznienie. Przez ulamek sekundy na wyjsciu pojawiaja sie smieciowe, posrednie wartosci (to jest **hazard**).

Flip-flop rozwiazuje problem - czeka az logika sie "uspokoi", a potem na zboczu zegara lapie stabilny wynik:

```
Zbocze CLK     Zbocze CLK     Zbocze CLK
    │              │              │
    ▼              ▼              ▼
 [lap dane] → [obliczaj] → [lap wynik] → [obliczaj] → ...
```

Bez tego CPU próbowalby czytac wyniki zanim bramki skoncza obliczenia - i dostawalby losowe bzdury.

### Uproszczony cykl

Zegar synchronizuje wszystkie operacje. Na kazdym zboczu narastajacym (`posedge clk`) przerzutniki zatrzaskuja nowe wartosci:

1. Zbocze narastajace -> przerzutniki zatrzaskuja dane
2. Miedzy zboczami -> logika kombinacyjna oblicza nowe wartosci (propagacja sygnalow przez bramki)
3. Nastepne zbocze -> wyniki zostaja zatrzasniate

Dlatego `always @(posedge clk)` jest tak fundamentalne w Verilogu.

## Praktycznie - jak to zrobic

Na FPGA - masz oscylator na plytce + bloki PLL/MMCM:

```verilog
// Xilinx - uzycie wbudowanego PLL
// Na plytce np. Basys3 masz krysztal 100 MHz
// PLL moze wygenerowac inne czestotliwosci

module clock_divider (
    input  wire clk_in,   // 100 MHz z krysztalu na plytce
    output reg  clk_out   // podzielony zegar
);
    reg [23:0] counter = 0;

    always @(posedge clk_in) begin
        counter <= counter + 1;
        clk_out <= counter[23]; // ~6 Hz - widoczne na LED
    end
endmodule
```

Na breadboardzie - najprostszy oscylator to obwod z bramkami NOT (np. uklad 74HC04) lub dedykowany modul oscylatora (np. kwarc w obudowie z 4 pinami - VCC, GND, OUT, EN):

```
Modul oscylatora (np. 1 MHz)
┌──────────┐
│ VCC  OUT ├──-> sygnal zegarowy do CPU/FPGA
│ GND  EN  │
└──────────┘
```

Najtansze wejscie - kup plytke FPGA (np. Tang Nano 9K ~$15, albo Basys3) - maja wbudowany oscylator i PLL, wiec od razu mozesz eksperymentowac z zegarem w Verilogu bez lutowania.

## Kluczowe pojecia

| Pojecie | Znaczenie |
|---|---|
| Czestotliwosc | Ile cykli/sekunde (Hz) |
| Duty cycle | Stosunek czasu stanu wysokiego do okresu (zwykle 50%) |
| Clock domain | Grupa logiki na jednym zegarze |
| Clock skew | Roznica w czasie dotarcia zegara do roznych przerzutnikow |
| Setup/hold time | Minimalne czasy stabilnosci danych przed/po zboczu zegara |
