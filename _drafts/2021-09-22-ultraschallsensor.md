---
layout: post
title:  "Abstandsmessung mit dem Ultraschallsensor"
date:   2021-09-22
tags: Ultraschallsensor Raspberry
---

Mit einem Ultraschallsensor können Abstände gemessen werden. Verbreitet ist der Ultraschallsensor HC-SR04, ich nutze den HC-SR05. Die Nutzung ist ähnlich.

((TODO: Foto))

Der Ultraschallsensor HC-SR04 hat vier Anschlüsse:
* VCC: 5V
* GND
* Trigger
* Echo

Der HC-SR05 hat noch einen Anschluss mehr, wenn man diesen ungenutzt lässt, kann man ihn wie einen HC-SR04 nutzen.

Der Sensor funktioniert vereinfacht wie folgt:

Durch Setzen des Trigger-Pins auf 1 für 0.01ms verschickt der Sensor einen Puls von Ultraschallwellen. Werden diese von einem Objekt in einer bestimmten Entfernung reflektiert, erkennt ein Empfänger im Sensor dieses Echo. Danach geht liegt am Echo-Pin für einen bestimmten Zeitraum ein Wert von 1 a. Aus der Zeitdauer dieses Echo-Pulses kann dann mit Hilfe der Schallgeschwindigkeit der Abstand berechnet werden:

Entfernung = Laufzeit * Schallgeschwindigkeit / 2

Genauere Beschreibungen sind hier zu finden:
* http://www.netzmafia.de/skripten/hardware/RasPi/Projekt-Ultraschall/index.html
* https://www.mikrocontroller-elektronik.de/ultraschallsensor-srf05-fuer-entfernungsmessungen/


## Steuerung mit dem Raspberry Pi

### Aufbau und Komponenten

Man braucht:
* ein Raspberry Pi (z.B. Zero)
* ein Breadboard
* ein Widerstand 470 Ω
* ein Widerstand 330 Ω


![Schaltplan Ultraschallsensor](/images/fritzing_ultraschallsensor.png)


### Programmierung mit RPi.GPIO

In nachfolgendem Python-Programm kann man die einzelnen Schritte der Messung gut nachvollziehen.

```python
#Bibliotheken
import RPi.GPIO as GPIO
import time
 
#GPIO definieren (Modus, Pins, Output)
GPIO.setmode(GPIO.BCM)
GPIO_TRIGGER = 5
GPIO_ECHO = 6
GPIO.setup(GPIO_TRIGGER, GPIO.OUT)
GPIO.setup(GPIO_ECHO, GPIO.IN)
 
def entfernung():
    # Trig High setzen
    GPIO.output(GPIO_TRIGGER, True)
 
    # Trig Low setzen (nach 0.01ms)
    time.sleep(0.00001)
    GPIO.output(GPIO_TRIGGER, False)
 
    Startzeit = time.time()
    Endzeit = time.time()
 
    # Start/Stop Zeit ermitteln
    while GPIO.input(GPIO_ECHO) == 0:
        Startzeit = time.time()
 
    while GPIO.input(GPIO_ECHO) == 1:
        Endzeit = time.time()
 
    # Vergangene Zeit
    Zeitdifferenz = Endzeit - Startzeit
	
    # Schallgeschwindigkeit (34300 cm/s) einbeziehen
    entfernung = (Zeitdifferenz * 34300) / 2
 
    return entfernung
 
if __name__ == '__main__':
    try:
        while True:
            distanz = entfernung()
            print ("Distanz = %.1f cm" % distanz)
            time.sleep(1)
 
        # Programm beenden
    except KeyboardInterrupt:
        print("Programm abgebrochen")
        GPIO.cleanup()
``` 
### Programmierung mit gpiozero

Die Klasse `DistanceSensor`aus der gpiozero-Bibliothek nimmt einem hier Arbeit ab. Ein vergleichbares Programm sieht so aus:

```python
from gpiozero import DistanceSensor
from time import sleep

sensor = DistanceSensor(echo=6, trigger=5)

while True:
    print('Distance: ', sensor.distance * 100)
    sleep(1)
```

Zusätzlich hat man die Möglichkeit callback-Funktionen festzulegen die ausgeführt werden, wenn die Distanz unter einen bestimmten Wert sinkt, bzw. über diesen Wert steigt:

```python
from gpiozero import DistanceSensor
from time import sleep

def in_range_func():
    print("nah")

def out_range_func():
    print("fern")

sensor = DistanceSensor(echo=6, trigger=5, threshold_distance=.2)
sensor.when_in_range = in_range_func
sensor.when_out_of_range = out_range_func

pause()
```

TODO: pigpiod



## Steuerung mit dem Calliope

http://micoro.de/calliope-mit-ultraschallsensor-hc-sr04/

### Aufbau und Komponenten

### Programmierung



## Steuerung mit dem ESP32 

### Programmierung
