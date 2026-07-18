# Virtual Shooting Range

Interaktywny symulator balistyki zewnętrznej działający w przeglądarce — wizualizacja toru pocisku 3D, luneta z siatką, tabela poprawek (DOPE) i punktowana tarcza.

## Uruchomienie

1. Rozpakuj archiwum do jednego folderu.
2. Otwórz `Virtual Shooting Range.dc.html` w przeglądarce (Chrome / Edge / Firefox).
3. Plik `support.js` musi znajdować się w tym samym folderze — bez niego aplikacja nie wystartuje.

Aplikacja działa w pełni offline; nie wysyła żadnych danych.

## Funkcje

- **Silnik balistyczny** — integracja RK4, model oporu G1, poprawka na gęstość powietrza (temperatura, ciśnienie, wilgotność), wiatr z 12 kierunków (tarcza zegarowa), kąt strzału (nachylenie).
- **Presety amunicji** — .22 LR, 5.56 NATO, .308 Win i inne; pełna edycja Vo, masy (gr) i BC.
- **Zerowanie i poprawki** — wysokość lunety, dystans zerowania, korekty ELEV/WIND w MOA lub MIL.
- **Widok 3D strzelnicy** — tor pocisku, punkt trafienia, znaczniki dystansu, kamera swobodna / za strzelcem / z boku.
- **Widok przez lunetę** — siatka celownicza ze wskazaniem punktu trafienia.
- **Rodzaje tarcz** — pierścieniowa Ø 50 / Ø 100 cm (punktacja 1–10), gong stalowy Ø 30 cm, papierowa IPSC 45 × 57,5 cm (strefy A/C/D), własna płyta Ø 10–200 cm. Wynik trafienia w panelu NA CELU.
- **Tabela poprawek (DOPE)** — opad i znoszenie co 25/50 m w cm oraz MOA/MIL, prędkość, energia, czas lotu.
- **Eksport PDF** — przycisk „⎙ PDF" otwiera stronę wydruku A4 z parametrami i pełną tabelą (zapis przez okno drukowania przeglądarki). Wymaga zezwolenia na wyskakujące okna.
- **Języki** — polski i angielski; jednostki dystansu m/yd.

## Pliki

| Plik | Rola |
|---|---|
| `Virtual Shooting Range.dc.html` | Aplikacja — interfejs, silnik balistyczny, rendering |
| `support.js` | Biblioteka wykonawcza (wymagana) |
| `favicon.svg` | Ikona strony |

## Zastrzeżenie

Symulacja ma charakter poglądowy/edukacyjny. Wyniki nie zastępują rzeczywistego zerowania broni ani profesjonalnych kalkulatorów balistycznych.
