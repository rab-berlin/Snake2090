# Snake2090

Snake für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4 Tastern.

- 246 Programmschritte, 10 frei
- Datei: [snake.mic](snake.mic)

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

## Technik: Die Matrix ist der Spielstand

Das Programm hat keinen eigenen Bildspeicher. Das Bild liegt in den Speicherregistern M0-MF, genau so, wie es an die Matrix geht: Zeile y ist M(2y) links und M(2y+1) rechts, Bit 1 ist die linke LED. Die Schlange und das Futter sind dort einfach gesetzte Bits. Das Programm merkt sich nur:

- Kopf, Schwanz und Futter, je als Matrixregister (0-F) und Bitmaske (1, 2, 4, 8),
- die Länge,
- die Richtungsschlange (siehe unten).

| Register | Bedeutung |
|---|---|
| R0 / R1 | Kopf: Matrixregister / Bitmaske |
| R2 / R3 | Schwanz: Matrixregister / Bitmaske |
| R4 / R5 | Futter: Matrixregister / Bitmaske |
| R6 | Länge der Schlange (2-11) |
| R7-RC | Richtungsschlange Q0-Q5 |
| RD, RE, RF | Hilfsregister: Index, Ergebnis / Richtung, Maske |

Aus dem Bild allein lässt sich der Weg des Schwanzes nicht bestimmen. Deshalb speichert das Programm jede Richtung in einer Richtungsschlange: 6 Register mit 12 Plätzen zu je 2 Bit. Daraus ergibt sich die Höchstlänge 12.

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt F an und hält es, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Er setzt kein REQ und beachtet die Ausgänge nicht, so ist Eingang 1 frei für Taster links.

Weil das F stehen bleibt, bis der ESP antwortet, geht kein Wecken verloren, auch wenn er gerade mit der Weboberfläche beschäftigt ist.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP).

## Geschichte

- **1976:** *Blockade* von Gremlin Industries (San Diego) ist das früheste bekannte Spiel dieser Art. Zwei Spieler ziehen jeder eine wachsende Linie hinter sich her und dürfen weder Wand noch eine Linie berühren. Im selben Jahr erschien *Bigfoot Bonkers* von Meadows Games, 1977 *The Surround* für das Atari VCS (Allan Miller).
- **1982:** *Snake Byte* von Chuck Sommerville (Sirius Software) kennt zum ersten Mal Futter: Die Schlange wächst, wenn sie Äpfel frisst. Das ist die Form, die wir heute meinen und die auch dieses Programm spielt.
- **1997:** Der Nokia-Entwickler **Taneli Armanto** schrieb *Snake* für das Nokia 6110. 2000 folgte *Snake II* auf dem Nokia 3310. Für viele wurde es das erste Spiel auf dem Handy.
- Es gibt Dutzende Varianten und kein Urheberrecht auf die Idee. Dieses Programm folgt dem Prinzip von *Snake Byte*: Futter fressen, wachsen, nicht in sich selbst laufen.

Quellen: snakegame.net „About Snake“ (Angaben dort nicht gegengeprüft).

## Wie es gemacht ist

Der 2090 rechnet nur mit ganzen Zahlen von 0 bis 15, kennt keine indirekte Adressierung und hat keinen XOR-Befehl. Ein paar Kniffe, mit denen Snake in 246 Schritte passt:

### Eine Zelle umschalten: ein Unterprogramm für alles

`Zelle` schaltet das Bit IDX/MASKE in der Matrix um und liefert den alten Wert zurück. Damit erledigt eine einzige Routine das Setzen des Kopfes, das Löschen des Schwanzes, das Setzen des Futters und die Kollisionsprüfung:

- **Umschalten ohne XOR:** Der 2090 hat kein XOR. Es gilt aber a XOR m = (a ODER m) − (a UND m). Das Programm rechnet T = a UND m, a = a ODER m und a = a − T. Ist das Bit in `m` vorher gesetzt, wird es gelöscht, sonst gesetzt.
- **Kollision gratis:** T ist nach dem Umschalten der alte Wert der Zelle. Das letzte `MOV T,T` setzt daraus das Zero-Flag: Null bedeutet, die Zelle war frei. Setzt der Kopf eine Zelle, die schon belegt war, trifft er die Schlange. Das Programm springt dann ins Spielende. Beim Futter wird ein Treffer genauso erkannt: Es setzt das Futter, war die Zelle belegt, schaltet es zurück und würfelt neu.

### Fenster auf die Matrix

Ein Register mit einer Nummer aus einem Register gibt es nicht. Das Programm legt deshalb mit `EXRL` die Matrixzeilen 1-4 (M0-M7) nach R0-R7, mit `EXRA, EXRM, EXRA` die Zeilen 5-8 (M8-MF). Der Index wird mit einem Vergleichsbaum aus `CMPI` und `BRC`/`BRZ` halbiert, bis das richtige Register R0-R7 gefunden ist. Danach schließt dieselbe Folge das Fenster wieder. Die Variablen des Spiels (R0-RF) wandern beim Tausch kurz in den Speicher und kommen unverändert zurück.

### Ein Schritt in vier Richtungen, und der Rand

`Schritt` schiebt IDX/MASKE ein Feld weiter und erkennt den Rand am Carry, ohne etwas zu vergleichen:

- **Links/rechts:** `SHR` bzw. `SHL` auf die Maske. Fällt das Bit heraus (Carry), ist die Registergrenze erreicht. Dann gibt es das Nachbarregister, außer die Schlange war schon in der linken bzw. rechten Hälfte am Rand. Das steht im niedrigsten Bit des Registerindex.
- **Hoch/runter:** Der Registerindex steigt oder fällt um 2. Läuft er über 15 oder unter 0, zeigt das der Carry: Rand, Spielende.

### Die Richtungsschlange

Nur die Matrix zu kennen reicht nicht, um den Schwanz nachzuziehen: Man sieht nicht, in welche Richtung die Schlange an ihrem Ende gelaufen ist. Die Richtungsschlange ist ein Schieberegister der letzten 12 Züge zu je 2 Bit (0 links, 1 hoch, 2 runter, 3 rechts), verteilt auf Q0-Q5:

- Platz j liegt in Q(j mod 6), für j < 6 in den Bits 1-2, für j ≥ 6 in den Bits 3-4.
- **Platz 0** ist die Vorwahl: Der Taster schreibt dorthin, der Zug selbst liegt erst danach auf Platz 1.
- Bei jedem Zug rückt alles einen Platz weiter. Platz 5 geht dabei in die obere Hälfte von Q0 (Platz 6), alles andere sind einfache `MOV`.
- Die Richtung am Schwanz steht auf dem Platz mit der Nummer LÄNGE. Wieder gibt es keine indirekte Adressierung: Der Platz wird durch den Vergleichsbaum `Ld`-`L5` ausgewählt, die obere Hälfte durch zweimal `SHR` herausgeschoben.

Frisst die Schlange, bleibt der Schwanz stehen und die Länge steigt um 1. Länge 12 füllt die Richtungsschlange vollständig: Das ist der Sieg.

### Die Taster

Die vier Taster liegen an den Eingängen 1-4, ein einziges `DIN` liest alle. Nach rechts geschoben (`SHR`) zeigt der Carry den ersten gedrückten: Er ist die Richtung, gezählt von 0. Der niedrigste Taster gewinnt, also muss das Programm nicht entscheiden, wenn zwei Tasten zugleich gedrückt sind.

Die Gegenrichtung wird ohne Fallunterscheidung erkannt: Die Richtungen sind so numeriert, dass links + rechts = 0 + 3 und hoch + runter = 1 + 2 jeweils 3 ergeben. Ist die Summe aus dem letzten Zug und der neuen Wahl 3, wird die Wahl ignoriert.

Die Taster werden zweimal je Zug gelesen, einmal vor dem Kopf und einmal nach dem Schwanz. Das ist der Grund, warum man die Taste halten soll, bis die Schlange abbiegt.

### Futter würfeln

`RND` liefert in den Registern D, E und F drei Ziffern. D ist das Matrixregister (0-F), E bestimmt die Spalte im Register (E mod 4). Die Maske wird durch eine kleine Schleife aus Verdopplungen (1, 2, 4, 8) gebildet. Weil `RND` D, E und F überschreibt, liegen dort nur Hilfswerte, die gerade nicht gebraucht werden. Ist die Zelle belegt, wird neu gewürfelt.

### Spielende

Die Länge steht im Register, `HXDZ` wandelt sie nach dezimal, `DISP 2` zeigt sie an. Danach hält das Programm in einer Endlosschleife. Länge 12 ist der Sieg.
