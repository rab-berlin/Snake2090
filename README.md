# Snake2090

Snake für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4 Tastern.

- 246 Programmschritte, 10 frei
- Datei: [snake.mic](snake.mic) (Version 06)

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

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt F an und hält es, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Er setzt kein REQ und beachtet die Ausgänge nicht, so ist Eingang 1 frei für Taster links.

Weil das F stehen bleibt, bis der ESP antwortet, geht kein Wecken verloren, auch wenn er gerade mit der Weboberfläche beschäftigt ist.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Das Programm braucht die Firmware mit „ein Befehl je Wecken“ (esp2090-firmware ab PR #38).
