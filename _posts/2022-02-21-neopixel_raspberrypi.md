---
layout: post
title:  "Neopixel am Raspberry Pi"
date:   2022-02-21 
tags: Neopixel Raspberry Levelshifter
---

Neopixel kann man auch am Raspberry Pi betreiben, detaillierte Infos sind bei Adafruit zu finden: [https://learn.adafruit.com/neopixels-on-raspberry-pi](https://learn.adafruit.com/neopixels-on-raspberry-pi)

Hilfreich sind auch folgende Seiten:
* [https://dordnung.de/raspberrypi-ledstrip/ws2812](https://dordnung.de/raspberrypi-ledstrip/ws2812)
* [https://github.com/jgarff/rpi_ws281x](https://github.com/jgarff/rpi_ws281x)


Danach ist Folgendes zu beachten:
* Je nach dem, wieviele Neopixel man nutzt, genügt die Stromversorgung über den Raspberry Pi nicht aus.
* Da die Neopixel eigentlich ein 5V-Signal benötigigen, kann der Einsatz eines Level-Shifters nötig sein.
* Zur Ansteuerung kann man die Circuit-Python-Neopixel-Bibliothek verwenden.
* Bei der Nutzung sind root-Rechte erforderlich.
* Neopixel müssen mit einem der folgenden Pins verbunden werden: GPIO10, GPIO12, GPIO18 oder GPIO21. GPIO18 ist der Standard Pin, GPIO10 hat bei mir nicht richtig funktioniert.

## Verkabelung ohne Levelshifter

Ohne externe Stromversorgung und ohne Level-Shifter kann man den 12er-Neopixel-Ring einfach wie folgt anschließen:

* Raspi 5V an Neopixel 5V
* Raspi GND an Neopixel GND
* Raspi GPIO-Pin 18 an Neopixel Din


## Verkabelung mit Levelshifter

Die GPIO-Pins des Raspberry Pi sind für 3,3 Volt ausgelegt, der Neopixel benötigt aber eigentlich 5 Volt. Für die Umwandlung kann man einen Levelshifter verweden, z.B. den Debo KY-051 Voltage Translator.

Die Anleitung hierzu gibt es hier:
[https://joy-it.net/files/files/Produkte/COM-KY051VT/COM-KY051VT-Anleitung.pdf](https://joy-it.net/files/files/Produkte/COM-KY051VT/COM-KY051VT-Anleitung.pdf)

Die Verkabelung sieht dann wie folgt aus:
* Raspi 3,3 Volt an Levelshifter VCCa
* Raspi GPIO-Pin 19 an Levelshifter A1
* Raspi GND an Breadboard GND
* externe 5 Volt Stromversorgung an Breadboard 5V
* externe Stromversorgung GND an Breadboard GND
* Breadboard 5V an Levelshifter VCCb
* Breadboard GND an Levelshifter GND
* Levelshifter B1 an Neopixel(-Ring)
* Neopixel Power an Breadboard 5V
* Neopixel GND an Breadboard GND

![Schaltplan mit Levelshifter](/images/fritzing_raspi_levelshifter_neopixel.png)

Diese Verkabelung hat allerdings bei mir nicht mit den 5V vom Raspi sondern nur mit externer Stromversorgung funktioniert.

## Programmierung mit Python

Für die Programmierung der Neopixel mit Python gibt es folgende Bibliotheken:
* `rpi_ws281x`
* `adafruit-circuitpython-neopixel`

Die Bibliothek `adafruit-circuitpython-neopixel` baut auf `rpi_ws281x` und auf `adafruit-blinka` auf. Sie wurde von Adafruit für Circuit-Python entwickelt und steht durch `adafruit-blinka` auch für den Raspberry Pi zur Verfügung.

Alternativ kann man die Bibliothek `rpi_ws281x` auch direkt nutzen.


### Programmierung mit adafruit-circuitpython-neopixel

Vorraussetzung ist die Installation der nötigen Bibliotheken:

```
$ sudo pip3 install rpi_ws281x adafruit-circuitpython-neopixel
$ sudo pip3 install adafruit-blinka
```

Um die Neopixel - im Beispiel einen Ring mit 12 LED - zu nutzen, erzeugt man zunächst ein Objekt der Klasse Neopixel. Dabei gibt man den GPIO-Pin sowie die Anzahl der Pixel an. 

Anschließend kann man auf einzelne LEDs zugreifen, indem man deren Position als Index angibt. Die Farben kann man entweder als RGB-Tupel (z.B. `(255,255, 0)`) oder aber als Hexadezimalwert (z.B. `0xFFFF00`) angeben.


```python
import board
import neopixel
from time import sleep

# erzeuge Objekt für NeoPixel-Ring mit 12 LED an GPIO-Pin 18
ring = neopixel.NeoPixel(board.D18, 12)
print(ring.auto_write)

# allgemeine Helligkeit ändern
ring.brightness= .1

# einzelne Pixel ansteuern
ring[0] = (0, 0, 255)
sleep(2)
ring[5] = (255, 255, 0)
ring[6] = 0xFFFF00
sleep(2)

# gesamten Ring mit bestimmter Farbe füllen
ring.fill((0, 255, 0))
sleep(2)
ring.fill((255, 0, 0))
sleep(2)
ring.fill((0, 0, 0))
```

Die API der Circuitpython-Neopixel-Bibliothek ist hier zu finden: [https://circuitpython.readthedocs.io/projects/neopixel/en/latest/](https://circuitpython.readthedocs.io/projects/neopixel/en/latest/)

### Programmierung mit rpi_ws281x

Will man die Bibliothek `rpi_ws281x` direkt nutzen, kann man sich an folgendem Beispielcode orientieren: [https://raw.githubusercontent.com/rpi-ws281x/rpi-ws281x-python/master/examples/strandtest.py](https://raw.githubusercontent.com/rpi-ws281x/rpi-ws281x-python/master/examples/strandtest.py)

Dabei kann man die Anzahl der Pixel `LED_COUNT` und die Helligkeit `LED_BRIGHTNESS` an die eigenen Bedürfnisse anpassen.
