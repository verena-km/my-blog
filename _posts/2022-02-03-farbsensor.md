---
layout: post
title:  "Farbsensor TCS34725"
date:   2022-02-03
tags: Farbsensor I2C Raspberry
---

* TOC
{:toc}


Zur Erkennung von Farben kann man einen Farbsensor verwenden, beispielsweise den "Debo Sens Color" mit dem TCS34725-Chip.

![Foto Farbsensor](/images/foto_farbsensor.jpg)

Der Sensor enthält ein Fotodioden-Array und ermittelt Werte für Rot, Grün, Blau und Ungefiltert (clear). Die Werte werden vom Sensor als 16-Bit-Werte bereitgestellt. Auf den Sensor kann über I2C zugegriffen werden. Er hat eine integrierte LED zur Beleuchtung.

Der Sensor hat folgende sieben Anschlüsse:
* LED: Steuerung der integrierten LED
* Int: Interrupt
* SDA: I2C data
* SCL: I2C Clock
* 3V3: Stromversorgung 3.3 Volt
* GND: Ground
* VIN: Stromversorgung 5 Volt

Zur Nutzung benötigt man auf jeden Fall eine Stromversorgung, GND sowie SDA und SCL.

Über den LED-Eingang kann man mit die integrierte LED steuern. Standardmäßig ist die LED an. Verbindet man den LED-Eingang mit Ground, ist die LED aus, verbindet man sie mit einem digitalen Pin, kann man darüber steuern, ob die LED an oder aus ist.

Der Sensor verfügt zudem über ein Interrupt-Feature. Der Nutzer kann dabei Schwellwerte definieren und sich über den Interrupt-Pin informieren lassen, wenn die Messung innerhalb dieser Schwellwerte ist.

Der optimale Abstand des zu messenden Objekts hängt von dessen Größe, der Beleuchtung etc. ab. Am besten ist es, wenn man das Objekt möglichst direkt vor dem Sensor platziert.

Die gemessenen Werte können über die Einstellungen von Verstärkung (Gain) und "Integration Time" an die jeweiligen Umstände (Lichtverhältnisse etc.) angepasst werden. Für eine Messung werden die Sensoren für eine bestimmte Zeit (die Integration-Time) angeschaltet. Je länger die Zeit ist, desto mehr Licht wird erkannt. Die Sensitivität der Sensoren kann über die Verstärkung verändert werden. Eine höherer Gain-Wert verstärkt die gelesenen Werte. Ein Messwert von 65535 mit einer Verstärkung von 1 und einer Integrationszeit von 2,4 Millisekunden ist also viermal so hell wie ein Messwert von 65535 mit einer Verstärkung von 4. Ebenso ist ein Messwert von 65535 mit einer Integrationszeit von 2,4 ms zehnmal so hell wie ein Messwert von 65535 mit einer Integrationszeit von 24 ms.

Das Datenblatt für den Sensor ist [hier](https://cdn-shop.adafruit.com/datasheets/TCS34725.pdf) zu finden.


## Anschluss an den Raspberry Pi

Der Sensor wird über I2C am Raspi angeschlossen.
* Raspi Pin 3 an SDA
* Raspi Pin 5 an SCL
* 3.3 V an 3.3V
* GND an GND

Für die Programmierung mit dem Raspberry Pi gibt es folgende Möglichkeiten:
* Verwendung der Circuit-Python-Bibliothek adafruit-circuitpython-tcs34725
* direkte Nutzung mit smbus

## Nutzung Adafruit Blinka

Vom Unternehmen Adafruit gibt es ein Fork von MicroPython namens CircuitPython. Adafruit stellt CircuitPython für eine Reihe von Microcontroller-Boards zur Verfügung. Für viele Elektronikkomponenten - auch für den TCS34725 - hat Adafruit Bibliotheken für CircuitPython entwickelt, so auch für den TCS34725.

Damit man Circuit-Python-Bibliotheken auch mit dem "normalen" Python auf dem Raspberry Pi nutzen kann, benötigt man aber zusätzlich die Bibliothek `Adafruit-Blinka`

Mit folgenden Befehlen kann man Adafruit-Blinka und die Circuit-Python-Bibliothek für den Farbsensor auf dem Raspberry Pi installieren:

```
$ sudo pip3 install Adafruit-Blinka
$ sudo pip3 install adafruit-circuitpython-tcs34725
```

Danach kann man in Python-Programmen auf die Sensordaten wie folgt zu greifen

```python
import board
import adafruit_tcs34725
from time import sleep


i2c = board.I2C()
sensor = adafruit_tcs34725.TCS34725(i2c)

while True:

    print('Color: ({0}, {1}, {2}, {3})'.format(*sensor.color_raw))
    print('Color: ({0}, {1}, {2})'.format(*sensor.color_rgb_bytes))
    print('Temperature: {0}K'.format(sensor.color_temperature))
    print('Lux: {0}'.format(sensor.lux))
    sleep(2)
```

Folgende Werte können als Properties vom Sensor gelesen werden:

* color_raw - ein 4-Tupel mit den 16-Bit-Werten für Rot, Grün, Blau und Clear
* color_rgb_bytes - ein Tupel mit den Farbwerten für Rot, Grün und Blau (jeweils zwischen 0 und 255)
* color_temperatur - die Farbtemperatur in Kelvin (berechnet)
* lux - die Beleuchtungsstärke

Die folgenden Properties können gelesen und geändert werden um das Verhalten des Sensors zu ändern:
* integration_time - Die Integrationszeit in Millisekunden, ein Wert zwischen 2.4 und 614.4.
* gain - Der Verstärkungsfaktor, kann die Werte 1, 4, 16 oder 60 annehmen.

Über die Property `color_rgb_bytes` erhält man aus den 16-Bit-Farbwerten des Sensors ein Tupel mit RGB-Byte-Werten. Der Wert ist bezogen auf den "clear"-Wert normalisiert und enthält eine Gamma-Korrektur.

Weitere Informationen gibt es bei Adafruit und in der entsprechenden API-Dokumentation:

* [https://learn.adafruit.com/adafruit-color-sensors/python-circuitpython](https://learn.adafruit.com/adafruit-color-sensors/python-circuitpython)
* [https://circuitpython.readthedocs.io/projects/tcs34725/en/latest/api.html](https://circuitpython.readthedocs.io/projects/tcs34725/en/latest/api.html)
* [https://github.com/adafruit/Adafruit_CircuitPython_TCS34725/blob/main/adafruit_tcs34725.py](https://github.com/adafruit/Adafruit_CircuitPython_TCS34725/blob/main/adafruit_tcs34725.py)

Heise-Artikel zu Circuit-Python auf dem Raspi
[https://m.heise.de/make/artikel/Ausprobiert-CircuitPython-auf-dem-Raspberry-Pi-4436550.html?seite=all](https://m.heise.de/make/artikel/Ausprobiert-CircuitPython-auf-dem-Raspberry-Pi-4436550.html?seite=all)


## Nutzung direkt mit smbus

Will man Adafruit-Blinka nicht nutzen, kann man auch mit der smbus-Bibliothek mit dem Sensor kommunizieren. 

Beispielcode dafür gibt es unter [http://www.pibits.net/amp/code/raspberry-pi-and-tcs34725-color-sensor.php](http://www.pibits.net/amp/code/raspberry-pi-and-tcs34725-color-sensor.php)

Mit diesem Beispielcode erhält man die Rohdaten des Sensors und man kann die Werte für die "Gain" und "Integration-Time" verändern.

## Praktische Nutzung

Je nach Aufgabenstellung will man zumeist
* vorgegebene Farben oder Farbbereiche erkennen oder
* einen Farbwert ermitteln, der dann in anderer Form (LED, Bildschirm) ausgegeben wird.

Zur Erkennung bestimmter Farben kann man in einem Kalibrierungvorgang deren Farbwerte bestimmen und unter ihrem Namen abspeichern. Später kann man dann gemessene Werte mit diesen abgespeicherten Werten vergleichen und den Namen der Farbe zurückgeben, deren abgespeicherter Wert dem gemessenen Wert am nächsten ist. Hierzu kann man die euklidische Distanz berechnen.

Durch die Veränderung von "Gain" und "Integration-Time" kann man die Helligkeitswerte der Messergebnisse verändern. Wählt man die Werte zu hoch, können bei hellen Farben keine Unterschiede mehr erkannt werden (z.B. zwischen weiss und hellgelb). Wählt man die Werte zu niedrig, werden die Farben zu dunkel.

Auch bei angepassten Werten von "Gain" und Integration Time" zeigt sich in der Praxis, dass die gemessenen RGB-Werte bei dunklen Farben dunkler und bei hellen Farben heller sind als die tatsächlichen Farben. Damit hat man bei der Ausgabe am Bildschirm Abweichungen gegenüber der tatsächlichen Farbe. 

Für meine Test habe ich farbige Papierstücke genutzt und diese jeweils 2 mm oberhalb des Sensors platziert.

![Foto Farbsensor Test](/images/foto_farbsensor_messung.jpg)

## Beispiel 1: Erkennung bestimmter vorher festgelegter Farbwerte

Mit folgendem auf der Adafruit-Bibliothek basierendem Programm kann man zunächst interaktiv bestimmte Farbnamen und Werte einlesen. Gibt man "q" ein, werden die Werte abgespeichert. Danach wird der Sensorwert alle zwei Sekunden ermittelt und angezeigt. Zu dem Sensorwert wird aus der eingelesenen bzw. gespeicherten Liste die ähnlichste Farbe ermittelt und deren Name ausgegeben.

```python
from turtle import color
import board
from adafruit_tcs34725 import TCS34725
from time import sleep
import math
import json


def euclid_dist(color1, color2):
    """ Euclidian distence between two 3-tuple
    in Python 3.8 we can use math.dist()"""
    sum = (color1[0]-color2[0])**2 + (color1[1]-color2[1])**2 + (color1[2]-color2[2])**2
    return math.sqrt(sum)


class MyBlinkaTCS34725(TCS34725):
    """ Modified adafruit base class, use list of safed colors to get nearest color name"""   
    def __init__(self):
        i2c = board.I2C()
        super().__init__(i2c)
        self.colorfilename = 'color_list.json'
        self.gain = 4
        self.integration_time = 200
        self.color_list = {}

        # read color list from file
        try:
            f = open(self.colorfilename)
            self.color_list = json.loads(f.read())
            f.close()
        except FileNotFoundError:
            print('Keine Farbliste gespeichert.')

    def read_colors(self):
        """Start interactive dialog to read color names and save data in file""" 
        end = False
        while not end:
            colorname = input(f"Bitte Gegenstand vor den Sensor halten und Farbname eingeben, Ende mit 'q':")
            if colorname == "q":
                end = True
            else:
                self.color_list[colorname] = self.color_rgb_bytes
                print(self.color_list[colorname])
        self.safe_colors()
        
    def safe_colors(self):
        """Save color list to file"""
        with open(self.colorfilename, 'w', encoding='utf-8') as f:
            json.dump(self.color_list, f, ensure_ascii=False, indent=4)
    
    def has_safed_colors(self):
        """Check if there are saved color values"""
        return len(self.color_list) != 0

    def get_best_matching_color(self, sensor_value):
        """get name of best matching color for sensor_value"""
        if self.has_safed_colors():
            min_dist = 500
            for color_name, color_value in self.color_list.items():
                dist = euclid_dist(color_value,sensor_value)
                if dist < min_dist:
                    min_dist = dist
                    min_color_name = color_name
            return(min_color_name)

        else:
            #Keine Farbliste gespeichert
            return("")


if __name__ == "__main__":

    sensor = MyBlinkaTCS34725()
    if not sensor.has_safed_colors():
        sensor.read_colors()

    try:
        while True:
            value = sensor.color_rgb_bytes
            print('Color: ({0}, {1}, {2})'.format(*value))
            print(sensor.get_best_matching_color(value))
            sleep(2)
    except KeyboardInterrupt:
        print('interrupted!')

```

## Beispiel 2: Ausgabe der vom Sensor gemessen Farbe im Terminal

Um Farben im Terminal anzuzeigen kann man das Python-Package [sty](https://sty.mewo.dev/index.html) verwenden. Auszugebende Strings kann man damit mit den für die jeweilige Farbe erforderlichen Escape-Sequenz versehen.

Mit folgendem Programm kann man sich unter Nutzung der Adafruit-Bibliothek die vom Sensor gemessenen Farbwerte als Farbe im Terminal anzeigen lassen:

```python
import board
import adafruit_tcs34725
from time import sleep
import math
from sty import bg, ef, fg, rs

i2c = board.I2C()
sensor = adafruit_tcs34725.TCS34725(i2c)
sensor.gain = 4
sensor.integration_time = 350
while True:
    r,g,b = sensor.color_rgb_bytes
    print(bg(r,g,b)+ "     " + bg.rs + str(sensor.color_rgb_bytes))
    sleep(2)
```

Die Ausgabe sieht dann wie folgt aus:

![Screenshot Farbsensor](/images/screenshot_farbsensor.png)



## Ähnliche Projekte

[https://www.youtube.com/watch?v=o9W8wYJofuM](https://www.youtube.com/watch?v=o9W8wYJofuM)

[https://www.makerblog.at/2015/01/farben-erkennen-mit-dem-rgb-sensor-tcs34725-und-dem-arduino/](https://www.makerblog.at/2015/01/farben-erkennen-mit-dem-rgb-sensor-tcs34725-und-dem-arduino)








