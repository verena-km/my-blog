---
layout: post
title:  "Geekservo"
date:   2021-02-01
tags: Servomotor Motor Lego
---

Zufällig bin ich auf die folgenden "Geekservo"-Motoren gestoßen, die man prima für Lego-Robotik-Projekte verwenden kann, denn die Motor-Achsen sind Lego-Technic-Achsen und die Motoren können über Noppen bzw. Verbinder mit Lego bzw. Lego-Technic verbunden werden.

![Geekservos](/images/foto_geekservo.jpg)

Es gibt drei verschiedene Geek-Servos:

## Roter Geekservo

Der rote Geekservo (Geekservo Motor) ist eigentlich gar kein Servomotor, sondern ein Gleichstrommotor, der eine Spannung zwischen 3,3 und 6 Volt benötigt. Er hat zwei Anschlüsse:
* Rot für die Versorgungsspannung von 5 V
* Schwarz für Ground

Er kann direkt an eine Spannungsquelle angeschlossen oder über einen Motortreiber genutzt werden.

## Grauer Geekservo

Der graue Geekservo (Geekservo 9g 270° Servo) ist ein 270-Grad-Servomotor und benötigt ebenfalls eine Spannung zwischen 3,3 und 6 Volt. Er wird mit drei Kabeln angeschlossen:
* Rot für die Versorgungsspannung 
* Braun für Ground
* Orange für das Steuersignal

Er kann wie übliche Servomotoren genutzt werden, wie in meinem [Post zu Servomotoren]({% post_url 2020-11-19-servo-motor %}) beschrieben. Gut geeignet ist der beispielsweise für die Lenkung eines Fahrzeugs.

## Grüner Geekservo

Der grüne Geekservo (Geekservo 9g 360° Servo) ist ein dauerhaft drehender Servomotor und benötigt ebenfalls eine Spannung zwischen 3,3 und 6 Volt. Er wird ebenfalls mit drei Kabeln angeschlossen:
* Rot für die Versorgungsspannung 
* Braun für Ground
* Orange für das Steuersignal

Mit der Pulsweite des PWM-Steuersignal kann man die Richtung und Geschwindigkeit des Motors verändern, dabei gilt:

* 500-1500 µs: Motor dreht rechts - je niedriger, desto schneller
* 1500 µs: Motor aus
* 1500-2500s:  Motor dreht links - je höher, desto schneller

Hier ein Testprogrammm für den Calliope Mini, bei dem mit den Tasten A und B die Pulsweite verändert und damit Geschwindigkeit bzw. Richtung geändert wird.
```javascript
input.onButtonPressed(Button.A, function () {
    pulsweite = pulsweite + 50
    pins.servoSetPulse(AnalogPin.P1, pulsweite)
})
input.onButtonPressed(Button.AB, function () {
    pulsweite = 0
    pins.servoSetPulse(AnalogPin.P1, pulsweite)
})
input.onButtonPressed(Button.B, function () {
    pulsweite = pulsweite - 50
    pins.servoSetPulse(AnalogPin.P1, pulsweite)
})
let pulsweite = 0
pulsweite = 1500
pins.servoSetPulse(AnalogPin.P1, pulsweite)
```
Das Tolle ist: Dieser 360°-Servo funktioniert ohne zusätzliche Spannungsquelle bei 3,3 Volt und ohne Motortreiber - in beide Richtungen und in unterschiedlichen Geschwindigkeiten. Also ideal für ein Mini-Robot-Car.