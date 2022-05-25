---
layout: post
title:  "Raspberry Pi Pico - Microcontroller"
date:   2022-04-29
tags: Microcontroller
---

## Allgemeines

Der Raspberry Pi Pico ist im Gegensatz zu den anderen Raspberry Pis kein Einplatinencomputer, sondern ein Microcontrollerboard wie der ESP32. Der Microcontroller ist vom Typ RP2040. Gegenüber dem ESP32 hat er allerdings den Nachteil, dass er nicht über WLAN und Bluetooth verfügt. Details zum Raspberry Pi Pico gibt es unter [https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html](https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html), im [Datasheet](https://datasheets.raspberrypi.com/pico/pico-datasheet.pdf) und [https://projects.raspberrypi.org/en/projects/getting-started-with-the-pico/0](https://projects.raspberrypi.org/en/projects/getting-started-with-the-pico/0)

Viele ausführliche Infos auf deutsch gibt es auch beim [Elektronik-Kompendium](https://www.elektronik-kompendium.de/sites/raspberry-pi/2604131.htm).

Der Raspberry Pi Pico kann in C/C++ oder in Micropython programmiert werden. 


## Hardware

Es gibt den Raspberry Pi Pico in Varianten mit bzw. ohne angelötete Pin-Leisten. Hier ein Modell mit Stiftleisten, das gut auf einem Breadboard platziert werden kann.

((Foto))

Über Micro-USB kann der Pico mit Spannung versorgt und auch programmiert werden. Die Spannungsversorgung ist auch über den VSYS-Eingang möglich, dort kann man eine Spannung von 1,8 bis 5,5 Volt anlegen. Intern verwendet der Pico 3,3 Volt. An VBUS liegen die 5V vom USB-Eingang an.

Auf dem Pico befindet sich eine Onboard-LED, z.B. für Tests und Statusanzeigen und ein mit BOOTSEL beschrifteter Taster.

Der Pico hat 26 GPIO-Pins:
* 2 mal SPI
* 2 mal I2C
* 2 mal UART
* 3 mal 12-Bit Analog-Digital-Wandler
* 16 PWM-Kanäle

Nachfolgend die Pin-Belegung des Raspberry Pi Pico:

![Raspberry Pi Pico Pinout](https://www.raspberrypi.com/documentation/microcontrollers/images/Pico-R3-SDK11-Pinout.svg)


## Programmierung mit Micropython

Um den Pico zu programmieren, benötigt man einen PC. Die Programme werden dann über USB auf den Pico übertragen. Im wesentlichen funktioniert das gleich wie beim ESP32, nur die Übertragung der Micropython-Firmware läuft etwas anders.

Detaillierte Informationen zur Programmmierung mit Micropython findet man in der offiziellen [Dokumentation des Raspberry Pi Pico Python SDK](https://datasheets.raspberrypi.com/pico/raspberry-pi-pico-python-sdk.pdf).

Zunächst muss man die Micropython-Firmware auf den Pico übertragen. Dann kann man sich über die serielle Schnittstelle mit ihm verbinden und auf ihm Python-Programme im interaktiv (im sog. REPL) ausführen.

Programmcode für den Pico kann man mit einem beliebigen Editor auf dem PC erstellen und dann auf den Pico übertragen. Die Möglichkeiten sind ähnlich wie beim ESP32:
* Mit der Python-IDE Thonny kann man Code erstellen, auf den Pico übertragen und auch auf den REPL zugreifen. Zudem kann man mit Thonnny auch die Micropython-Firmware auf den Pico übertragen
* Mit der VSCode-Erweiterung Pico-Go kann man den Programmcode in VS-Code erstellen und auf den Pico übertragen
* mit dem Kommandozeilentool ampy kann man Programmcode, der in einem beliebigen Editor erstellt wurde auf den Pico übertragen.


### Installation von Thonny

Unter Linux kann man Thonny über den Paketmanager installieren, z.B. unter Ubuntu mit
```
$ sudo apt-get install thonny
```

Unter Ubuntu ist dies allerdings eine ältere Version von Thonny. Installiert man über pip, erhält man die neuste Version, mit der man ach die Micropython-Firmware auf den Pico übertragen kann.

### Inbetriebnahme 

Um den Pico mit Micropython programmieren zu können, muss zunächst die Micropython-Firmware auf dem Board installieren. Dazu sind folgende Schritte durchzuführen.

1. Download der [UF2-Datei](https://micropython.org/download/rp2-pico/rp2-pico-latest.uf2)

2. BOOTSEL-Knopf getdrückt halten und Raspi einstecken

3. Der Raspi-Pico wird als USB-Device angezeigt

4. Die UF2-Datei in das Raspi-Pico-Laufwerk kopieren

Alternativ kann man dies auch mit Thonny machen, sofern man die neueste Version installiert hat:

Man geht im Menü auf "Run" - "Select Interpreter und wählt oben MicroPython (Raspberry Pi Pico) aus und klickt anschließend unten rechts auf "Install or update firmware" und folgt den angegebenen Schritten.

((screenshot))


### Zugriff über serielle Schnittstelle

Mit picocom kann man sich über die serielle Schnittstelle mit dem Pico verbinden und dort interaktiv Python-Kommandos eingeben um z.B. die interne LED an und wieder ausschalten.

```
$ picocom -b 115200 /dev/ttyACM0
picocom v3.1

...

Type [C-a] [C-h] to see available commands
Terminal ready

>>> from machine import Pin
>>> led = Pin(25, Pin.OUT)
>>> led.on()
>>> led.off()
>>> 

```

### Dateien auf dem Pico

Nutzt man Micropython können auf dem Raspberry Pi Pico Dateien gespeichert werden, insbesondere Dateien mit Python Code. Eine besondere Bedeutung dabei hat die Datei main.py. Der Code darin wird ausgeführt, sobald der Pico mit Strom versorgt wird. Programme können so ausgeführt werden, ohne dass der Pico mit einem PC verbunden ist. 

Es können auch Ordnerstrukturen angelegt werden. Das Filesystem des Pixo hat eine Größe von etwa 1,4 MB.


### Verwaltung der Dateien mit ampy

Mit ampy kann man über die Kommandozeile auf das Micropython-Dateisystem des Raspberry Pi Pico zugreifen. 

Zunächst muss man ampy mit pip installieren:

```sh
sudo pip install adafruit-ampy
```
Mit folgendem Befehl kann sich dann die auf dem ESP32 gespeicherten Dateien auflisten lassen.

```sh
ampy --port /dev/ttyACM0 ls
```

Wir erstellen nun eine Datei blink_pico.py, die die interne LED des Pico zum Blinken bringt:
```python
import time
from machine import Pin
interne_led = Pin(25, Pin.OUT)
while True:
    interne_led.on()
    time.sleep_ms(500) 
    interne_led.off()
    time.sleep_ms(500) 
```


Folgender Befehl führt diese Datei auf dem Pico aus:
```sh
ampy --port /dev/ttyACM0 run blink_pico.py
```
Folgender Befehl überträgt diese Datei auf den ESP32:
```sh
ampy --port /dev/ttyACM0 put blink_pico.py
```

Auf diesem Wege kann man auch die Datei main.py auf dem Computer mit einem beliebigen Editor erstellen und anschließend mit ampy auf den ESP32 übertragen.

Weitere Informationen zu ampy findet man hier: [https://github.com/scientifichackers/ampy](https://github.com/scientifichackers/ampy)


### Programmierung mit Thonny

Mit der MicroPython-Integration von Thonny kann man komfortabel auf das Dateisystem und den REPL von Micropython auf dem Raspberry Pi Pico zugreifen. Unter Run - Select Interpreter kann man den Interpreter auswählen. Dort wählt man MicroPython (Raspberry Pi Pico) und kann dann den Port auswählen.


### Programmierung mit VSCode

Für VSCode gibt die Extension [Pico-Go](http://pico-go.net). 

Nach Installation der Extension in VSCode erstellt man ein Verzeichnis für die Projektdateien und öffnet dieses in VSCode. 

Funktioniert aktuell nicht! 30.04.2022


## State Machine
https://community.hobbyelektroniker.ch/wbb/index.php?thread/336-statemachines-und-pio-programmierung-beim-raspberry-pi-pico/&postID=2875

https://www.heise.de/developer/artikel/I-O-on-Steroids-PIO-die-programmierbare-Ein-Ausgabe-des-Raspberry-Pi-Pico-6018818.html