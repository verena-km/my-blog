---
layout: post
title:  "Digitaler Schallsensor"
date:   2021-09-16
tags: Schallsensor
---

Der Schallsensor "Debo Sens Sonic" ist ein digitaler Sensor, der die Umgebungslautstärke erkennen kann. Man kann den Schwellwert auf dem Sensor mit einem kleinen Schraubenzieher einstellen. 

Der Schallsensor hat folgende Anschlüsse:
* OUT - digitaler Ausgang
* GRD
* VCC - 3,3 V bis 5 V

Wenn die Lautstärke kleiner ist als der eingestellte Schwellenwert, liegt am Ausgang eine 1 an, ansonsten eine 0.

Der Ausgang des Schallsensors kann man mit einem GPIO-Eingang des Microcontrollers oder Raspis verbinden. 

Nachfolgend ein Beipspielprogramm für den ESP32 mit dem Sound-Sensor an Pin 15:

```python
from machine import Pin
import time

sound_sensor = Pin(15, Pin.IN)
interne_led = Pin(2, Pin.OUT)
interne_led.off()

while True:
    if sound_sensor.value() == 0:
        print("geklatscht")
        if interne_led.value() == 0:
            interne_led.on()
        else:
            interne_led.off()
        time.sleep_ms(100)
```

Bei einem Geräusch (z.B. Klatschen) wird die interne LED des ESP2 angeschaltet und beim nächsten Geräusch wieder ausgeschaltet.

