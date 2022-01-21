---
layout: post
title:  "Calliope-Mini-Robot-Car mit Fernsteuerung"
date:   2021-10-22 
tags: Calliope Servomotor Geekservo Fernsteuerung 
---

* TOC
{:toc}

Klein und beweglich ist dieses ferngesteuerte Calliope-Fahrzeug. Das Fahrgestell ist aus Lego Technic, für den Antrieb werden zwei grüne 360-Grad-Geek-Servomotoren verwendet. Die Steuerung übernimmt ein Calliope Mini. Ferngesteuert wird mit einem zweiten Calliope Mini und dessen integrierten Neigungssensor. 

![Callipe Minicar](/images/foto_calliope_minicar.jpg)

Es ist damit die kleinere Variante des [mit dem Calliope gesteuerten frühren Prototyp]({% post_url 2020-09-19-calliope_steuert_calliope %}).

Die Calliope-Stromversorgung mit den 2 AAA-Batterien war für das Fahrzeug zu schwach, daher hatten wir diese zunächst mit einer 9-Volt-Batterie und dem Stepdown-Konverter realisiert. Die Batterien waren aber schnell leer. Nun kommt ein 18650 Li-Ion-Akku in einem Expansion Shield zum Einsatz.

## Komponenten und Aufbau

Man braucht dafür:
* 1 18650 Li-Ion-Akku
* 1 18650 Battery Expansion Shield
* 2 grüne Geek-Servo-Motoren (360-Grad-Servos)
* 1 USB-Kabel
* 2 Calliope Mini
* 1 Calliope Mini Batteriebox mit 2 AAA-Batterien
* und diverse Lego-Technic-Teile

Das Fahrzeug hat zwei von den Geek-Servos angetriebene Räder und ein Stützrad.

Über das Expansion Shield wird der Calliope Mini über USB mit Strom aus dem 18650 Li-Ion-Akku versorgt.
Der Calliope steurt über zwei Ausgänge die grünen Geek-Servos ([siehe auch Post zu den Geekservos]({% post_url 2021-02-01-geekservo %})), die die Räder antreiben.

![Schaltplan Mini-Robot-Car](/images/fritzing_calliope_minicar2.png)


## Programmierung

TODO: Programme überarbeiten

Es werden zwei Programme benötigt. Eins für den Calliope am Fahrzeug (Empfänger) und eins für die Fernbedienung.




Der Javascript-Code aus Makecode für das Fahrzeug sieht wie folgt aus:

```javascript
input.onGesture(Gesture.TiltRight, function () {
    radio.sendNumber(3)
})
radio.onReceivedNumber(function (receivedNumber) {
    basic.setLedColor(0xffff00)
    if (receivedNumber == 12) {
        basic.setLedColor(0x00ff00)
        basic.pause(5000)
        basic.turnRgbLedOff()
    }
})
input.onGesture(Gesture.LogoUp, function () {
    radio.sendNumber(0)
})
input.onGesture(Gesture.ScreenDown, function () {
    radio.sendNumber(-1)
})
input.onGesture(Gesture.TiltLeft, function () {
    radio.sendNumber(2)
})
input.onGesture(Gesture.ScreenUp, function () {
    radio.sendNumber(1)
})
basic.setLedColor(0xff0000)
radio.setGroup(173)
radio.sendNumber(12)
```


Man kann ihn per Copy & Paste in MakeCode einfügen und dann auf den Calliope übertragen.

Die Fernsteuerung läuft mit folgendem Javascript-Code:

```javascript
radio.onReceivedNumber(function (receivedNumber) {
    if (receivedNumber == 12) {
        basic.setLedColor(0x00ff00)
        radio.sendNumber(0)
    }
    if (receivedNumber == 0) {
        servos.P0.stop()
        servos.P3.stop()
    }
    if (receivedNumber == 1) {
        servos.P0.run(100)
        servos.P3.run(-100)
    }
    if (receivedNumber == -1) {
        servos.P0.run(-100)
        servos.P3.run(100)
    }
    if (receivedNumber == 2) {
        servos.P0.run(100)
        servos.P3.run(100)
    }
    if (receivedNumber == 3) {
        servos.P0.run(-100)
        servos.P3.run(-100)
    }
})
basic.setLedColor(0xff0000)
radio.setGroup(173)
```
