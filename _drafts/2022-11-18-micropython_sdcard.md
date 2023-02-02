---
layout: post
title:  "SD-Karte über SPI ansteuern"
date:   2022-11-18
tags: Micropython SD-Karte SPI
---

Da auf Microcontrollern selbst nur wenige Daten gespeichert werden können, ist es hilfreich, wenn man auch auf ein externes Speichermedium zugreifen kann. Mit einem SD-Card-Adapter-Modul kann man SD-Karten über das SPI-Protokoll mit Microcontrollern verbinden.

# Raspberry Pi Pico

## Hardwareaufbau

Wir verwenden folgende Komponenten:

* Raspberry Pi Pico
* DEBO MICROSD 2 SD-Karten-Modul
* 16 GB SD-Karte

Das SD-Karten-Modul hat folgende Anschlüsse:
* GND
* VCC
* MISO
* MOSI
* SCK
* CS

Beim Pico werden die folgenden Bezeichnungen verwendet

SPIx_SCK = SCK
SPIx_TX = MOSI
SPIx_RX = MISO
SPIx_CSn = CS

SPI1_RX  - MISO - GP8   Nr.11
SPI1_SCn - CS   - GP9   Nr.12
SPI1_SCK - SCK  - GP10  Nr.14
SPI1_TX  - MOSI - GP11  Nr.15

Wichtig ist, dass das VCC der Karte mit der richtigen Spannung verbunden ist, bei uns mit 5V (VBUS) 

(( Fritzing Verkabelung))

## Formatierung der SD-Karte

Die SD-Karte sollte zuvor mit einem FAT32-Dateisystem formatiert werden.

## Programmierung in Micropython

Die Micropython-Programmbibliothek für die Ansteuerung des SD-Karten-Moduls findet man unter [https://github.com/micropython/micropython-lib/blob/master/micropython/drivers/storage/sdcard/sdcard.py](https://github.com/micropython/micropython-lib/blob/master/micropython/drivers/storage/sdcard/sdcard.py). Diese legt man dann auf dem Microcontroller im Unterverzeichnis lib ab.

Für den Zugriff auf die SD-Karte erzeugt man zunächst ein Objekt für den CS-Pin und ein Objekt der Klasse SPI. Bei diesem muss man unter anderem die Pins für SCK, MOSI und MISO angeben. Diese beiden Objekte werden dann verwendet, um ein Objekt der Klasse SDCard zu erzeugen.

Danach wird das zur SD-Karte gehörende Filesystem-Objekt vfs erzeugt und unter im Verzeichnis /sd eingehängt (gemounted). Anschließend können Dateien und Verzeichnisse erzeugt, beschrieben, gelesen und gelöscht werden.

```python
import machine
import sdcard
import uos

# Assign chip select (CS) pin (and start it high)
cs = machine.Pin(9, machine.Pin.OUT)

# Intialize SPI peripheral (start with 1 MHz)
spi = machine.SPI(1,
                  baudrate=1000000,
                  polarity=0,
                  phase=0,
                  bits=8,
                  firstbit=machine.SPI.MSB,
                  sck=machine.Pin(10),
                  mosi=machine.Pin(11),
                  miso=machine.Pin(8))

# Initialize SD card
sd = sdcard.SDCard(spi, cs)

# Mount filesystem
vfs = uos.VfsFat(sd)
uos.mount(vfs, "/sd")

# Create a file and write something to it
with open("/sd/test02.txt", "w") as file:
    file.write("Hello, SD World!\r\n")
    file.write("This is a test\r\n")

# Open the file we just created and read from it
with open("/sd/test02.txt", "r") as file:
    data = file.read()
    print(data)


Weitere Infos:
[https://www.digikey.de/en/maker/projects/raspberry-pi-pico-rp2040-sd-card-example-with-micropython-and-cc/e472c7f578734bfd96d437e68e670050](https://www.digikey.de/en/maker/projects/raspberry-pi-pico-rp2040-sd-card-example-with-micropython-and-cc/e472c7f578734bfd96d437e68e670050)


## Nutzung der SD-Karte mit dem ESP32 mit Micropython

https://www.engineersgarage.com/micropython-esp32-microsd-card/

## Nutzung der SD-Karte mit dem ESP32 in der Arudino IDE


Programmiert man den ESP32 mit der Arduino-IDE, kann man das SD-Karten-Modul wie folgt anschließen


CS an GPIO5
SCK an GPIO18
MOSI an GPIO23
MISO an GPIO19
VCC an VCC
GND an GND


https://www.youtube.com/watch?v=9cLhrjc6pZY 

https://randomnerdtutorials.com/esp32-microsd-card-arduino/
https://github.com/espressif/arduino-esp32/tree/master/libraries/SD

Wichtig: 
an VCC anschließen, nicht an 3V
https://forum.arduino.cc/t/esp32s-cant-detect-mount-microsd-card/620150/3

## Nutzung der SD-Karte mit dem Arduino Uno

https://create.arduino.cc/projecthub/electropeak/sd-card-module-with-arduino-how-to-read-write-data-37f390 


https://github.com/arduino-libraries/SD