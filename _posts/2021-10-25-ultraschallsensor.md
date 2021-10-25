---
layout: post
title:  "Abstandsmessung mit dem Ultraschallsensor"
date:   2021-10-25
tags: Ultraschallsensor Raspberry
---

* TOC
{:toc}

Mit einem Ultraschallsensor können Abstände gemessen werden. Verbreitet ist der Ultraschallsensor HC-SR04.

Er hat vier Anschlüsse:
* VCC: 5V
* GND
* Trigger
* Echo


Ich nutze den HC-S05. Er hat noch einen Anschluss mehr, man ihn aber wie einen HC-SR04 nutzen.

Der Sensor funktioniert vereinfacht wie folgt:

Durch Setzen des Trigger-Pins auf 1 für 0,01 ms verschickt der Sensor einen Puls von Ultraschallwellen. Werden diese von einem Objekt in einer bestimmten Entfernung reflektiert, erkennt ein Empfänger im Sensor dieses Echo. Danach geht liegt am Echo-Pin für einen bestimmten Zeitraum ein Wert von 1 an. Aus der Zeitdauer dieses Echo-Pulses kann dann mit Hilfe der Schallgeschwindigkeit der Abstand berechnet werden:
```
Entfernung = Laufzeit * Schallgeschwindigkeit / 2
```

Eine genauere Beschreibung ist beispielsweise hier zu finden:
* [http://www.netzmafia.de/skripten/hardware/RasPi/Projekt-Ultraschall/index.html](http://www.netzmafia.de/skripten/hardware/RasPi/Projekt-Ultraschall/index.html)



## Steuerung mit dem Raspberry Pi

### Aufbau und Komponenten

Man braucht:
* ein Raspberry Pi (z.B. Zero)
* den Ultraschallsensor
* ein Breadboard
* ein Widerstand 470 Ω
* ein Widerstand 330 Ω

![Schaltplan Ultraschallsensor](/images/fritzing_raspi_ultraschallsensor.png)

Die Widerstände werden für einen Spannungsteiler benötigt. Dadurch wird sichergestellt, dass der GPIO-Eingang des Raspi nicht mehr als 3,3 V bekommt.

Die maximal an GPIO-Eingang anliegende Spannung kann man wie folgt berechnen:

```
U = 5 V / (470 Ω + 330 Ω) * 470 Ω = 3 V
```

### Programmierung mit RPi.GPIO

In nachfolgendem Python-Programm kann man die einzelnen Schritte der Messung gut nachvollziehen.

```python
import RPi.GPIO as GPIO
import time
 
GPIO.setmode(GPIO.BCM)
GPIO_TRIGGER = 5
GPIO_ECHO = 6
GPIO.setup(GPIO_TRIGGER, GPIO.OUT)
GPIO.setup(GPIO_ECHO, GPIO.IN)
# Timeout - wie lange wird auf Anfang bzw. Ende des Echo-Pulses gewartet
timeout = .04
 
def get_distance():
    # TRIGGER auf High
    GPIO.output(GPIO_TRIGGER, True)
 
    # Trigger auf Low (nach 0.01ms)
    time.sleep(0.00001)
    GPIO.output(GPIO_TRIGGER, False)

    # Warten auf Echo-Beginn
    pulse_start = time.time()
    timeout_end = pulse_start + timeout
   
    while GPIO.input(GPIO_ECHO) == 0 and pulse_start < timeout_end:
        pulse_start = time.time()
 
    # Warten auf Echo-Ende
    pulse_end = time.time()
    timeout_end = pulse_end + timeout

    while GPIO.input(GPIO_ECHO) == 1 and pulse_end < timeout_end:
        pulse_end = time.time()
 
    # Dauer Echo-Puls berechnen
    pulse_duration = pulse_end - pulse_start
	
    # Entfernung über Schallgeschwindigkeit (34300 cm/s) einbeziehen
    distance = (pulse_duration * 34300) / 2
 
    return distance
 
if __name__ == '__main__':
    try:
        while True:
            distanz = get_distance()
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

Zusätzlich hat man die Möglichkeit callback-Funktionen festzulegen, die ausgeführt werden, wenn die Distanz unter einen bestimmten Wert sinkt, bzw. über diesen Wert steigt:

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

Nutzt man gpiozero, sollte man für bessere Messergebnisse den `pigpiod` aktivien. Falls er noch nicht installiert ist, muss man ihn zunächst installieren
```
sudo apt-get install pigpio
```

Dann kann man ihr aktivieren:
```
sudo systemctl start pigpiod
```

Damit er immer gleich nach dem Systemstart läuft, kann man ihn enablen:
```
sudo systemctl enable pigpiod
```


## Steuerung mit dem ESP32 

Man braucht:
* ein ESP32 Entwicklerboard
* den Ultraschallsensor
* ein Breadboard
* ein Widerstand 470 Ω
* ein Widerstand 330 Ω

![Schaltplan Ultraschallsensor](/images/fritzing_esp32_ultraschallsensor.png)

Die Widerstände werden für einen Spannungsteiler benötigt. Dadurch wird sichergestellt, dass der GPIO-Eingänge des ESP-32 nicht mehr als 3,3 V bekommt.

Die maximal an GPIO-Eingang anliegende Spannung kann man wie folgt berechnen:

U = 5 / (470 Ω + 330 Ω) * 470 Ω = 3 V

### Programmierung

Die Programmierung kann man ähnlich wie beim Raspberry Pi machen:

```
from machine import Pin
import time
 
trigger_pin = Pin(33, Pin.OUT)
echo_pin = Pin(32, Pin.IN)

# Timeout - wie lange wird auf Anfang bzw. Ende des Echo-Pulses gewartet
timeout_us = 40000
 
def get_distance():

    trigger_pin.value(0) # Stabilize the sensor
    time.sleep_us(5)

    # TRIGGER auf High
    trigger_pin.value(1)
 
    # Trigger auf Low (nach 0.01ms)
    #time.sleep(0.00001)
    time.sleep_us(10)
    trigger_pin.value(0)
 
    # Warten auf Echo-Beginn
    pulse_start = time.ticks_us()
    timeout_end = pulse_start + timeout_us
   
    while echo_pin.value() == 0 and pulse_start < timeout_end:
        pulse_start = time.ticks_us()
 
    # Warten auf Echo-Ende
    pulse_end = time.ticks_us()
    timeout_end = pulse_end + timeout_us

    while echo_pin.value() == 1 and pulse_end < timeout_end:
        pulse_end = time.ticks_us()
       
    # Dauer Echo-Puls berechnen
    pulse_duration = (pulse_end - pulse_start) / 1000000 # Wert in Sek

    # Entfernung über Schallgeschwindigkeit (34300 cm/s) einbeziehen
    distance = (pulse_duration * 34300) / 2
 
    return distance
 

while True:
    distanz = get_distance()
    print ("Distanz = %.1f cm" % distanz)
    time.sleep(1)

```

Alternativ kann man die fertige Klasse HCSR04 aus folgendem Github Repository verwenden:

* [https://github.com/rsc1975/micropython-hcsr04](https://github.com/rsc1975/micropython-hcsr04)


## Steuerung mit dem Calliope

Da der Calliope keinen 5V-Ausgang hat und mein Ultraschallsensor 5V benötigt, kann man ihn leider nicht direkt am Calliope betreiben.

Lösungmöglichkeiten sind:
- Nutzung eines Ultraschallsensors für 3,3 Volt
- externe Stromversorgung für den Ultraschallsensor
- Einatz eines Level-Shifters