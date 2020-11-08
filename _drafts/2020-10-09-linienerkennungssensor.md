---
layout: post
title:  "Linienerkennungssensor"
date:   2020-10-09
tags: Sensor 
---


Zum Bau eines Linienverfolgers nutze ich den Iduino 1485324 Linien-Erkennungssensor. 

((Bild))

Er hat drei Pins:

* S: digitaler Ausgang
* V+: Spannung 3,3V bis 5 V
* Ground

An seinem digitalen Ausgang liegt bei "Weiß" eine 0 und bei "Schwarz" eine 1 an.

Verbindet man den Ausgang S de Sensors mit dem GPIO-Pin 15 des ESP32 kann man sich die Sensorwerte ausgeben lassen:

```python 
from machine import Pin
import time
linechecker = Pin(15, Pin.IN)

while True:
    print(linechecker.value())
    time.sleep_ms(50)
```