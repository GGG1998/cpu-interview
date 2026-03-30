# Narzędzia synchronizacji

- Omówienie problemu sekcji krytycznej, omówienie szkodliwej rywalizacji.
- Sprzętowe rozwiązania problemu sekcji krytycznej, takie jak bariery pamięciowe, operacje "porównaj i zamień",
- Przydatność muteksowych, semaforów, monitorów i zmiennych warunkowych do rozwiązania problemu sekcji krytycznej.
- Ocena narzędzi rozwiązywania problemu sekcji krytycznej w warunkach małej, umiarkowanej i dużej rywalizacji


## Słowniczek
Współbieność i równoległość

Proces współpracujący = taki, który może oddziaływać na inne procesy LUB na który inne procesy wywierają wpływ.

1. Bezpośrednie dzielenie logicznej przestrzeni adresowej                                                                                                                              

To są wątki (threads) w ramach jednego procesu. Wszystkie wątki procesu widzą 
tę samą przestrzeń adresową — ten sam kod, te same zmienne globalne, ten sam heap.

Nie trzeba niczego specjalnie konfigurować — wątek A pisze do zmiennej x, wątek B od razu to widzi, 
bo oba patrzą na ten sam adres w pamięci.
       
2. Współużytkowanie przez pamięć dzieloną / komunikaty
To są oddzielne procesy — każdy ma swoją własną, izolowaną przestrzeń adresową. 
Proces A nie widzi pamięci procesu B. Żeby się komunikować, muszą użyć  
specjalnego mechanizmu:
- Pamięć dzielona (shared memory) — OS mapuje ten sam fragment pamięci fizycznej 
do przestrzeni adresowej obu procesów (np. shmget/shmat w POSIX)       
- Przekazywanie komunikatów (message passing) — procesy wysyłają/odbierają wiadomości 
przez OS (np. potoki, sockety, kolejki komunikatów)    

## Notes Reference
Czy pamiętasz, że strumień instrukcji procesu może zostać przerwany?