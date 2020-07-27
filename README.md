# Java 

## Image blur

Damit eine Aufgabe sich gut für paralelle Programmierung/Multithreading eignet, ist es von Vorteil, wenn das Problem einfach in kleinere Teile zerlegt werden kann, die dann selbstständig und abhängig voneinander abgearbeitet werden können. 
Auf [Wikipedia](https://en.wikipedia.org/wiki/Embarrassingly_parallel) gibt es einen kurzen Artikel darüber, welche Probleme am besten für parallele Programmierung geeignet sind. 
Einige Anwendungen sind Beispiele aus der Bildverarbeitung. 

In diesem Beispiel gilt es eine Bild-Datei einzulesen und unkenntlich zu machen (blur Effekt). 
Diese Aufgabe kann perfekt zerlegt werden, da wir lediglich die Werte eines Pixels ändern abhängig von seinem Wert und n Nachbar Werten.
Das kann im Prinzip für alle Pixel gleichzeitig geschehen. 

Wie in der Invert Image Aufgabe solltest du einen `ForkJoinPool` verwenden, mit den Thresholds zum Splitten herumspielen und einige Zeitmessungen in einem `timing-results.md` dokumentieren.

### Logik zum Errechnen des neuen Pixel-Wertes 

Ein Pixel kann als Integer dargestellt werden (RGB). 
Unser Integer besteht aus 4 Bytes und kann als HEX folgendermaßen dargestellt werden 0xAABBCCDD.
AA repräsentiert das Byte, das die Helligkeit bzw. Dunkelheit der Farbe darstellt. 
BB repräsentiert das Byte, das den Rotwert angibt, 
CC repräsentiert das Byte, das den Grünwert angibt,
DD repräsentiert das Byte, das den Blauwert angibt.

Zum blurren unseres Bildes errechnen wir für jeden einzelnen Farbwert (RGB) den Durchschnitt über X Pixel nach links und rechts aus und setzen unseren Wert auf diesen.

## Bitmasking & Shifting [Advanced]

Anstelle der `java.awt.Color` Klasse können wir auch  mit der Integer/Byte repräsentation eines Pixels rechnen. 
Wenn es dir leichter fällt kannst du aber auch `.getGreen()`, `.getRed()` und `.getBlue()` verwenden. 

Um lediglich den R Wert eines RGB-Integers zu bekommen brauchen wir ein paar bitweise Operatoren: 
```
int pixel = 0x12AB45CD;
int red = (pixel & 0x00ff0000) >> 16; // results in 0x000000AB
int green = (pixel & 0x0000ff00) >> 8; // results in 0x00000045
int blue = pixel & 0x000000ff; // results in 0x000000CD
```

Das einfache `&` ist ein bitweises und, dadurch dass wir unseren Integer mit einer Maske verunden bekommen wir einen neuen Integer zurück bei dem nur jene Bytes gesetzt sind die in der Maske einen Wert haben. Ein Hex-Wert von 0xFF entspricht binär einer 0b11111111 - d.h. alle Bits sind gesetzt.
D.h. `0x12AB45CD & 0x00ff0000` liefert uns `0x00AB0000` und `0x12AB45CD & 0x0000ff00` liefert uns `0x00004500`. 
Die doppelten größer Zeichen `>>` sind der Shift right operator und er verschiebt den gesetzten Wert um die Anzahl der Bits nach rechts. Ein Byte hat 8 bits darum müssen wir um den rot Wert als Zahl extrahieren zu können den Wert um 2 Bytes, also 16 Bits verschieben. 
D.h. `0x00AB0000 >> 2*8` liefert uns `0x000000AB` und `0x00004500 >> 1*8` liefert `0x00000045`. 

Wenn du dann den durchschnittlichen Wert für Rot, Grün und Blau errechnet hast kannst du dir das Ergebnis Pixel wieder mit Bitmaskierungen und bitweisem Shiften zusammenbauen: 
```
int newPixel = 0xff000000 | (redAverage << 16) | (greenAverage << 8) | (blueAverage << 0);
```

Das einfache `|` ist ein bitweises oder. 
D.h. `0xFF000000 & 0x0000EE00` liefert uns `0xFF00EE00`.