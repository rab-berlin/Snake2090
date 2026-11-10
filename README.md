# Snake2090

Snake für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4 Tastern.

- 253 Programmschritte, 3 frei
- Datei: [snake.mic](snake.mic) (Version 05)

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

RESET (alle Register und Speicher 0, das Bild liegt in M0-MF), Matrix einschalten (siehe unten), `HALT` `NEXT` `00` `RUN`.

## Technik

Aus dem Bild allein lässt sich der Weg des Schwanzes nicht bestimmen. Deshalb speichert das Programm jede Richtung in einer Richtungsschlange: 6 Register mit 12 Plätzen zu je 2 Bit. Daraus ergibt sich die Höchstlänge 12.

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) (Blockmodus mit Schlafmodus):

- **Schlafen:** Zwischen den Bildern schläft die Matrix (ACK 15). Der ESP setzt dann kein REQ und ignoriert die Ausgänge, so ist Eingang 1 frei für Taster links.
- **Wecken und Bild:** Für ein Bild legt das Programm F an und hält es, bis der ESP mit REQ antwortet. Dann sendet es direkt ACK 1, genau einen Befehl und die 16 Register. So geht kein Wecken verloren, egal wie spät der ESP nachsieht (z. B. bei offener Weboberfläche).
- **Wieder schlafen:** Danach setzt das Programm die Ausgänge auf 0 und legt die Matrix wieder schlafen (ACK 15, nach zwei Schritten eine 0).

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Die Matrix muss vor dem Start eingeschaltet sein.

Bis Version 02 lief das Programm mit dem früheren Blockprotokoll (Startnibble 1, Endnibble 0, Umschaltsignal F).

Das Programm braucht die Firmware des ESP2090 Studios mit dem Weck-Handshake (gehaltenes F).
