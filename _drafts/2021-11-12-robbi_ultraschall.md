---
layout: post
title:  "Raspi-Robot-Car mit Ultraschallsensor"
date:   2021-11-12 
tags: Ultraschallsensor Raspberry Beispielfahrzeug
---

Das Raspi-Fahrzeug wird nun mit einem Ultraschallsensor ausgestattet, damit er rechtzeitig bevor er auf ein Hindernis trifft, die Richtung ändern kann.

Grundlage ist dieses [Basis-Fahrzeug]({%post_url 2021-09-21-raspi-car-gpiozero%}). Informationen zum Ultraschallsensor und dessen Ansteuerung sind [hier]({%post_url 2021-10-25-ultraschallsensor%}) zu finden.

## Einfache Variante

In der einfachen Variante wird der Ultraschallsensor vorne fest am Fahrzeug montiert:

![Foto Fahrzeug](/images/foto_raspi_ultraschallsensor_car.jpg)

Hierfür kleben wir das kleine Breadboard, auf das der Ultraschallsensor und die Widerstände gesteckt wurden,  auf eine Lego 4x4-Plate. Diese kann man dann mit Hilfe von 3/4-Pins mit Lego-Liftarmen verbinden und dann vorne am Fahrzeug anbringen.

![Foto Ultraschallsensor](/images/foto_ultraschallsensor_lego.jpg)


Den Ultraschallsensor mit Widerständen schließen wir dann wie hier [hier]({%post_url 2021-10-25-ultraschallsensor%}) beschrieben an den Raspi an.

Insgesamt sieht die Verkabelung dann so aus:

![Schaltplan Ultraschallsensor](/images/fritzing_raspi_ultraschallsensor_car.png)

Den Raspi kann man dann unter Verwendung von gpiozero wie folgt programmieren:

```python
from gpiozero import Robot
from gpiozero import DistanceSensor
from gpiozero.pins.pigpio import PiGPIOFactory
from time import sleep
from signal import pause


factory = PiGPIOFactory()
robot = Robot((24,23),(12,25))
sensor = DistanceSensor(echo=6, trigger=5, threshold_distance=.2, pin_factory = factory)

def turn_right():
    print("Hallo Wand")
    robot.right(.5)
    sleep(.5)
    robot.forward(.5)

sensor.when_in_range = turn_right

robot.forward(.5)
pause()
```
Fährt das Fahrzeug nun frontal in Richtung eines Hindernisses, wird dies erkannt, sobald der Abstand kleiner als 20 cm ist und das Fahrzeug biegt nach rechts ab.

Wenn das Fahrzeug allerdings schräg gegen ein Hindernis fährt, erkennt der Ultraschallsensor dies nicht. Es wäre also gut, wenn der Ultraschallsensor auch seitlich in verschiedenen Positionen die Abstände messen könnte.

## Variante mit beweglichem Ultraschallsensor

Hierfür setzen wir den Ultraschallsensor auf einen Servomotor und können dann die Abstände zu möglichen Hinternissen aus verschiedenen Positionen messen. Wir nutzen dafür den [grauen 270°-Geek-Servo]{%post_url 2021-02-01-geekservo%} und verbinden die Achse des mit einem runden 2x2 Legostein mit Achsloch (3941). Darauf kann man dann die 4x4-Plate mitsamt Minibreadboard und Ultraschallsensor setzen.

((Foto))

Den Geek-Servo schließen wir wie folgt an:

* Rotes Kabel an Versorgungsspannung 5V (auf dem Breadboard)
* Braunes Kabel an Ground (auf dem Breadboard)
* Oranges Kabel an Pin 21 am Raspberry Pi





