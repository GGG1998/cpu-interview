⚡ 1. Skąd bierze się prąd w gniazdku?

W gniazdku domowym masz prąd przemienny (AC) 230 V, 50 Hz (w Europie).
To oznacza:

napięcie zmienia się sinusoidalnie od +325 V do –325 V względem zera (wartość skuteczna to 230 V),
50 razy na sekundę przebieg przechodzi przez zero i zmienia kierunek.

W przewodach:
L (live, faza) — przewód „gorący”, w którym napięcie zmienia się względem neutralnego,
N (neutral) — przewód odniesienia, prawie na potencjale ziemi,
PE (protective earth, uziemienie) — przewód ochronny, łączony z bolcem w gniazdku i obudową metalowych urządzeń.

⚡ 2. Dlaczego zasilanie trzeba przekształcić?

Układy elektroniczne (np. logika 5 V, LED-y, mikrokontrolery) nie mogą być zasilane bezpośrednio z 230 V AC, bo:

napięcie jest za duże,

jest przemienne (zmienia polaryzację),

jest niebezpieczne dla człowieka i elektroniki.

Dlatego stosuje się zasilacze, które robią kilka rzeczy:

⚡ 3. Budowa typowego zasilacza (230 V → np. 5 V DC)

Transformator – obniża napięcie z 230 V AC do np. 12 V AC (i zapewnia separację galwaniczną).
Mostek prostowniczy (diody) – zamienia przebieg przemienny na jednokierunkowy (pulsujący DC).
Kondensator filtrujący – wygładza napięcie, robiąc prawie stałe DC.
Stabilizator napięcia – ustala dokładnie np. 5 V lub 3,3 V.
Schemat blokowy wygląda tak:

230V AC → [transformator] → [prostownik] → [kondensator] → [stabilizator] → 5V DC
