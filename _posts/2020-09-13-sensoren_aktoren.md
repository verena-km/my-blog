---
layout: post
title:  'Sensoren und Aktoren des Roboterfahrzeugs'
date:   2020-09-13 09:00:00 +0200
tags: motor sensor
---

## Motoren

Für die Bewegung des Roboters sind als "Aktor" vor allem Motoren wichtig. Bei den Motoren unterscheidet man zwischen
* Gleichstrommotoren (DC-Motoren)
* Servomotoren und
* Schrittmotoren

Gleichstrommotoren haben zwei Anschlüsse und drehen sich, wenn sie mit einer Spannungsquelle verbunden werden. Ändert man die Pole drehen sie sich in die andrere Richtung. Ich verwende die Lego M- bzw. L-Motoren. In folgender Abbildung sieht man links dem M-Motor und rechts den L-Motor:

![Foto Lego Motoren](/images/foto_lego_motoren.jpg)

Servomotoren kann man so ansteuern, dass sie eine bestimmte Winkelpositon einnehmen. Ich verwende einen einfachen Micro-Servomotor
* an den ich mit Heißkleber an zwei Seiten Lego-Platten 2x3 geklebt habe und
* an den ich oben eine 2x2-Platte geschraubt habe.

![Foto Servo-Motor](/images/foto_servo.jpg) 

Er hat drei Anschlüsse:
* 5 V - Versorgungsspannung
* Ground
* PWM-Signal für Positionierung

Den Schrittmotor hab ich bislang noch nicht eingesetzt.

## Sensoren

Folgende Sensoren möchte ich ausprobieren:

* Ultraschallsensor
    für Abstandsmessung

* Lage / Magnetfeld / Beschleunigungssensor
    für Richtungsbestimmung

* Speedsensor

und weitere für:
* Kollisionserkennung
* Linienfolger
* usw.


