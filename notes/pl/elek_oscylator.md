🔹 2. Oscylator (ang. oscillator)

To już kompletny generator sygnału zegarowego.

Składa się z rezonatora (np. kwarcu) + dodatkowych elementów elektronicznych (wzmacniacz, kondensatory, układ startowy).

Na wyjściu daje gotowy sygnał prostokątny, np. 16 MHz.

Zazwyczaj występuje w obudowie 4-nóżkowej (moduł oscylatora kwarcowego).

Takie oscylatory często stosuje się w komputerach PC, serwerach, FPGA – tam, gdzie potrzebna jest stabilność i prostota podłączenia.

## Dlaczego oscylator jest tak ważny?
Każda aplikacja wymagająca do pracy informacji o czasie korzysta z serwera czasu. Przeważnie sygnał czasu wzorcowego jest pobierany z satelity GPS lub bezpośrednio z zegara atomowego. Jednak w wyniku zakłóceń lub zaniku sygnału, czas wzorcowy może być nieprawidłowy lub niemożliwy do odebrania. W tym przypadku do akcji wkracza oscylator – wewnętrzny system synchronizacji i podtrzymywania czasu. Jego najważniejszym zadaniem jest tzw. holdover – utrzymanie prawidłowego czasu rzeczywistego do czasu przywrócenia sygnału czasu wzorcowego. Jest to niesamowicie odpowiedzialna i ważna praca – w branżach infrastruktury krytycznej sieć jest tak wysoce zależna od prawidłowego czasu zegara, że opóźnienie czasu rzędu μs czy ms może spowodować tylko małą i/lub krótką awarię, ale może również narazić przedsiębiorstwo milionowe straty np. poprzez uszkodzenie elementów sieci czy paraliż dostarczania usługi i związane z tym wypłaty odszkodowań.

https://interelcom.com/pl/baza-wiedzy/oscylatory-kwarcowe-czym-sa-i-jak-dzialaja-74286#:~:text=Oscylatorem%20kwarcowym%20nazywamy%20rezonator%20wraz,jako%20sygna%C5%82%20taktuj%C4%85cy.