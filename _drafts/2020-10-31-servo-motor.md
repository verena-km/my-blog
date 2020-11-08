---
layout: post
title:  "Servomotor"
date:   2020-10-31
tags: Servomotor 
---

Die Achse eines Servomotors kann man auf bestimmte Positionen meist in einem Bereich von 0 bis 180° drehen. Ein Servomotor ist daher für Drehbewegungen um maximal 180° auf genau bestimmte Zielpositionen geeignet. Das ist beispielsweise bei einer Lenkung der Fall.

Einfache Servomotoren wie der Y-3009 haben drei Anschlüsse:
* Rot für die Versorgungsspannung von 5 V
* Braun für Ground
* Orange für das Steuersignal

Für die Ansteuerung der jeweiligen Winkelposition wird Puls-Weiten-Modulation (PWM) genutzt und zwar ein Signal von 50 Hz, also eine Periodenlänge von 20 ms. Die Position des Servos ist abhängig von der Dauer des High-Pegels im PWM-Signal. Dabei gilt in der Regel:

* 1,5 ms von 20 ms = 7,5 % entspricht 0° (Mittelposition)
* High-Pegel zwischen 0,5 und 2,5 ms


Wenn das der High-Pegel kürzer ist, dreht sich der Servo im Uhrzeigersinn, wenn er länger ist, dreht er sich gegen den Uhrzeigersinn.

Welche Wert genau welchem Winkel entspricht, hängt vom jeweiligen Servo ab. Ebenso die Minimal und Maximalwerte. 

Für meinen Y-3009 gilt beispielsweise:

0,52 von 20 ms = Maximalausschlag im Uhrzeigersinn
2,33 von 20 ms = Maximalausschlag gegen den Uhrzeigersinn

0,56 von 20 ms = entspricht +90°
1,40 von 20 ms = entspricht 0°
2,33 von 20 ms = entspricht -90°

Für den MC-1811 gilt hingegen:

2,15 von 20 ms = Maximalausschlag im Uhrzeigersinn
0,8  von 20 ms = Maximalausschlag gegen den Uhrzeigersinn


0,6 ms von 20 ms = 3 % entspricht +90°
2,4 ms von 20 ms = 12,5 % entspricht -90° - allerdings mit komischem Brummen


## Ansteuerung unter Python mit dem Raspi

### Nutzung von RPi.GPIO


### Nutzung pigpio

Mit pigpio kann man einen Servo an Pin 18 wie folgt ansteuern:

```python
import pigpio
import time

# der (lokale) Raspi
pi = pigpio.pi()

# Servo an Pin 18
servo_pin = 18
# als Ausgang konfigurieren
pi.set_mode(servo_pin, pigpio.OUTPUT)

# Unterschiedliche Positionen ansteuern
# pos ist die Pulsweite in µs, Wert zwischen 500 und 2500 - 1500 ist die Mitte 
for pos in [500, 1000, 1500, 2000, 2500, 1500]:
    pi.set_servo_pulsewidth(servo_pin,pos) 
    time.sleep(1)
# beenden
pi.set_servo_pulsewidth(servo_pin,0)
pi.stop()
```

Zur Wiederverwendung kann man auch eine Klasse Servo definieren:

```python
class Servo:
    def __init__(self, servo_pin_nr):
        self.servo_pin_nr = servo_pin_nr
        self.pi = pigpio.pi()
        self.pi.set_mode(self.servo_pin_nr, pigpio.OUTPUT)

    def set_position(self,grad):
        # Werte abbilden
        # von -90 bis +90 auf 600 bis 2400
        if grad < -90 or grad > 90:
            pass
        else:
            position = (grad+90)*10+600
            self.pi.set_servo_pulsewidth(self.servo_pin_nr,position)
```

Diese kann man dann wie folgt nutzen:

servo = Servo(18)
servo.set_position(0)


## Ansteuerung unter Micropython mit dem ESP32




## Ansteuerung mit dem Calliope



Weitere Infos:

https://de.wikipedia.org/wiki/Servo