---
layout: post
title:  "Vererbung und Mehrfachvererbung in Python"
date:   2022-02-15
tags: Python
---

In objektorientierten Programmiersprachen können Klassen voneinander erben, so dass die Methoden und Attribute der Basisklasse auch in den abgeleiteten Klassen zur Verfügung stehen. Die abgeleiteten Klassen können dann zusätzliche Methoden und Attribute haben. Dadurch wird redundanter Code vermieden.

Ein Beispiel ist die Klasse Robot für ein Fahrzeug mit zwei Motoren. Diese hat die Methoden `forward`,`backward`, `right`, `left` und `stop`.

Stattet man das Fahrzeug noch mit einem Kompass-Sensor aus, kann man dafür eine weitere Klasse CompassRobot definieren, die von Robot erbt. Damit stehen in Objekten der Klasse CompassRobot ebenfalls die o.g. Methoden zur Verfügung. Zusätzlich kann man in der Klasse CompassRobot weitere Methoden definieren, wie beispielsweise `rotate_north` um das Fahrzeug in Richtung Norden zu drehen.

Stattet man ein Fahrzeug (bzw. die beiden Motoren) mit Speedsensoren aus, kann man aus der Klasse `Robot` die Klasse `SpeedsensRobot` ableiten, die dann als zusätzliche Methode beispielsweise das Vorwärtsfahren um eine bestimmte Strecke erhält.

Hat man nun ein Fahrzeug, das sowohl mit einem Kompasssensor, als auch mit Speedsensoren ausgestattet ist, kann man dafür eine Klasse `SpeedSensCompassRobot` die sowohl von `SpeedSensRobot` als auch von `CompassRobot` erbt. Eine solche Mehrfachvererbung ist in Python möglich.

Folgender Programmcode zeigt die prinzipielle Umsetzung (ohne Methoden): 
```python
class Robot():
    def __init__(self, motor_left, motor_right):
        print("Robot(): entering")
        self.motor_left = motor_left
        self.motor_right = motor_right
        print("Robot(): exiting")
    
class SpeedSensRobot(Robot):
    def __init__(self, s1, s2, **kwargs):
        print("SpeedSensRobot(): entering")
        super().__init__(**kwargs)
        self.s1 = s1
        self.s2 = s2
        print("SpeedSensRobot(): exiting")

class CompassRobot(Robot):
    def __init__(self, c1, c2, **kwargs):
        print("CompassRobot(): entering")    
        super().__init__(**kwargs)
        self.c1 = c1
        self.c2 = c2
        print("CompassRobot(): exiting")          

class SpeedSensCompassRobot(SpeedSensRobot, CompassRobot):
     def __init__(self, sc1, **kwargs):
         print("SpeedSensCompassRobot() entering")
         super().__init__(**kwargs)
         self.sc1 = sc1
         print("SpeedSensCompassRobot() exiting")

robot = SpeedSensCompassRobot(motor_left="motor1", motor_right="motor2", s1="s1", s2="s2", c1="c1", c2="c2", sc1="sc1")
print(f"{robot.sc1=}, {robot.s1=}, {robot.c1=}, {robot.motor_left=}, {robot.motor_right=}")
```

Wird nun ein Objekt der Klasse SpeedSensCompassRobot erzeugt, können dabei auch die für die Oberklassen erforderlichen Schlüsselwort-Parameter angegeben werden. Diese werden dann an im **kwargs-Dictionary an den Konstruktors der Superklassen weitergegeben und dort verarbeitet.

Durch die print()-Aufrufe sieht man, in welcher Reihenfolge die Konstruktoren aufgerufen werden:
```
SpeedSensCompassRobot() entering
SpeedSensRobot(): entering
CompassRobot(): entering
Robot(): entering
Robot(): exiting
CompassRobot(): exiting
SpeedSensRobot(): exiting
SpeedSensCompassRobot() exiting
```
Hier gilt die [Method Resolution Order (MRO) von Python](https://www.python.org/download/releases/2.3/mro/). Man kann sich diese auch anzeigen lassen:
```
print(SpeedSensCompassRobot.mro())
```

Weitere Infos zur Nutzung der Mehrfachvererbung sind hier zu finden:
* [https://stackoverflow.com/questions/3277367/how-does-pythons-super-work-with-multiple-inheritance](https://stackoverflow.com/questions/3277367/how-does-pythons-super-work-with-multiple-inheritance)