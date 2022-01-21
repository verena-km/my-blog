---
layout: post
title:  "Raspi-Robot-Car mit Ultraschallsensor"
date:   2021-11-12 
tags: Ultraschallsensor Raspberry Beispielfahrzeug
---

Das Raspi-Fahrzeug wird nun mit einem Ultraschallsensor ausgestattet, damit er,    rechtzeitig bevor er auf ein Hindernis trifft, die Richtung ändern kann.

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
Fährt das Fahrzeug nun frontal in Richtung eines Hindernisses, wird dies erkannt, sobald der Abstand kleiner als 20 cm. Es wird die Methode turn_right aufgerufen und das Fahrzeug fährt ein kleines Stück zurück und biegt nach rechts ab.

Wenn das Fahrzeug allerdings schräg gegen ein Hindernis fährt, erkennt der Ultraschallsensor dies nicht. Es wäre also gut, wenn der Ultraschallsensor auch seitlich in verschiedenen Positionen die Abstände messen könnte.

## Variante mit beweglichem Ultraschallsensor

Hierfür setzen wir den Ultraschallsensor auf einen Servomotor und können dann die Abstände zu möglichen Hinternissen aus verschiedenen Positionen messen. Wir nutzen dafür den [grauen 270°-Geek-Servo]({%post_url 2021-02-01-geekservo%}) und verbinden die Achse des mit einem runden 2x2 Legostein mit Achsloch (3941). Darauf kann man dann die 4x4-Plate mitsamt Minibreadboard und Ultraschallsensor setzen.

![Foto Fahrzeug mit Ultraschallsensor auf Servo](/images/foto_raspi_ultraaufservo_car.jpg)

Den Geek-Servo schließen wir wie folgt an:

* Rotes Kabel an Versorgungsspannung 5V (auf dem Breadboard)
* Braunes Kabel an Ground (auf dem Breadboard)
* Oranges Kabel an Pin 21 am Raspberry Pi

Damit können wir nun
* das Fahrzeug in verschieden Richtungen fahren lassen (in verschiedenen Geschwindigkeiten)
* den Ultaschallsensor in bestimmten Winkeln positionieren
* die Entfernungen messen
* bei der Unterschreitung bestimmter Entfernungen Aktionen ausführen

Um Hindernisse zu erkennen, lassen wir den Sensor während der Fahrt laufend von links nach rechts drehen und die Entfernungen messen. Dabei müssen Fahrgeschwindigkeit und Drehgeschwindigkeit des Sensors aufeinander abgestimmt sein. Die Zeit, die benötigt wird, um den Sensor einmal hin und her zu bewegen, sollte kleiner sein als die Zeit, die die Fahrt vom Erkennen des Hindernisses bis zum Erreichen des Hindernisses dauert.


```python
from gpiozero import Robot
from gpiozero import DistanceSensor
from gpiozero import AngularServo
from gpiozero.pins.pigpio import PiGPIOFactory
from time import sleep
from signal import pause

factory = PiGPIOFactory()
robot = Robot((24,23),(12,25))
sensor = DistanceSensor(echo=6, trigger=5, threshold_distance=.25, pin_factory = factory)
servo = AngularServo(21,min_angle=-145,
                          max_angle=145,
                          min_pulse_width=0.5/1000,
                          max_pulse_width=2.5/1000, 
                          pin_factory = factory)
speed = .4

def turn_right():
    print("Hallo Wand")
    robot.backward(speed)
    sleep(.5)
    robot.right(speed)
    sleep((1))
    robot.forward(speed)

def turn_sensor():
    angle = 0
    delay = 0.1
    step_angle = 20
    max_angle = 80
    while angle < max_angle:
        servo.angle = angle
        angle = angle + step_angle
        sleep(delay)
    while angle > -max_angle:
        servo.angle = angle
        angle = angle - step_angle
        sleep(delay)
    while angle < 0:
        servo.angle = angle
        angle = angle + step_angle
        sleep(delay)

sensor.when_in_range = turn_right
robot.forward(speed)

while True:
    turn_sensor()
```

Für den grauen Geek-Servo nutzen wir die Klasse AngularServo aus der gpiozero-Bibliothek. Die Funktion `turn_sensor` setzt die Hin- und Herbewegung um: Er startet in der Mitte, bewegt sich dann schrittweise bis zum maximalen Winkel (max_angle) und dann bis zum minimale Winkel (-max_angle) und dann wieder zur Mitte. Dabei kann man bei Bedarf folgendes anpassen:
* den maximalen Winkel max_angle
* den Winkel, um den pro Drehschritt weitergedreht wird
* die Zeit die bei jedem Drehschritt verweilt wird

Die Abstandsmessungen werden dabei in dem durch Nutzung von DistanceSensor erzeugten Thread durchgeführt - unabhängig von der jeweiligen Servoposition. Wenn der Schwellwert von 20 cm unterschritten wird, wir die Callback-Funktion turn_right aufgerufen.

Für weitere Optimierungen ist es jedoch besser, die Abstandsmessungen so durchzuführen, dass man die Messergebnisse den jeweiligen Servopositionen zuordnen kann. Daher wird in der nachfolgenden verbesserten Variante eine eigene Klasse SimpleDistanceSensor 


## Verbesserte Variante mit beweglichem Ultraschallsensor
