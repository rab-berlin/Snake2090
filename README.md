# Snake2090

Snake für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4 Tastern.

- 252 Programmschritte, 4 frei
- Datei: [snake.mic](snake.mic) (Version 02)

## Spiel

- **Start:** Die Schlange hat Länge 2 (Zeile 4, Spalte 3-4) und läuft nach rechts.
- **Futter:** Es erscheint zufällig auf einem freien Feld. Jedes Futterstück verlängert die Schlange um 1.
- **Spielende:** Kopf über den Rand oder auf die eigene Schlange. Die Anzeige zeigt die Länge dezimal.
- **Sieg:** bei Länge 12.
- **Tempo:** Ein Zug dauert etwa 1,4 s.

## Bedienung

| Taster | Pfeiltaste im Emulator | Richtung |
|---|---|---|
| Eingang 1 | links | links |
| Eingang 2 | hoch | hoch |
| Eingang 3 | runter | runter |
| Eingang 4 | rechts | rechts |

- Die Gegenrichtung wird ignoriert.
- Bei mehreren Tastern zählt der niedrigste.
- Den Taster halten, bis die Schlange abbiegt.

## Start

RESET (alle Register und Speicher 0, das Bild liegt in M0-MF), Matrix einschalten, `HALT` `NEXT` `00` `RUN`.

## Technik

Aus dem Bild allein lässt sich der Weg des Schwanzes nicht bestimmen. Deshalb speichert das Programm jede Richtung in einer Richtungsschlange: 6 Register mit 12 Plätzen zu je 2 Bit. Daraus ergibt sich die Höchstlänge 12.

## Matrixprotokoll

Das Programm verwendet das **frühere Blockprotokoll** der Matrix:
- Bereit-Signal auf Eingang 1
- Startnibble 1, 16 Werte, Endnibble 0
- ein F von 15-25 ms schaltet die Matrix um

Im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` auf `alt` stellen. Für das aktuelle Protokoll des [ESP2090 Studios](https://github.com/rab-berlin/ESP2090) muss das Programm noch umgestellt werden.
