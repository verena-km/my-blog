---
layout: post
title:  "Erster Prototyp mit Calliope"
date:   2020-07-19 17:08:49 +0200
tags: calliope Modell-A
---

((Foto))

Für den ersten Prototyp nehme ich den Calliope und dessen integrierten Motortreiber. Das Fahrzeug hat zwei angetriebene Räder und ein Stützrad. Aufgrund der Einschränkungen des Calliope-Motortreibers gibt es leider keinen Rückwärtsgang.

## Komponenten und Aufbau

Ich nutze folgende Komponenten:
* 1 Calliope Mini mit herausgeführter Pinleiste
* 2 Lego M-Motoren
* 1 Lego Power-Functions-Batteriebox
* 1 Step-Down-Converter

![Schaltplan Zwei-Motor-Betrieb](/images/fritzing_calliope_dual_motor.png)


Die Lego-Batteriebox wird mit den Eingängen des Step-Down-Konverters verbunden. Hierfür verwenden wir ein modifiziertes Lego-Verlängerungskabel und stecken den Lego-Stecker auf die Batteriebox. Am Eingang des Step-Down-Konverters schrauben wir jeweils bei "+" und bei "minus" zwei Jumper-Kabel-Verbindungstecker ein. Dananch verbindet man die rote Ader des Verbindungskabel mit einem Pin des mit "+" gekennzeichneten Eingangs, die schwarze Ader mit einem Pin des mit "-" gekennzeichneten Eingangs des Step-Down-Konverters. Das gelbe und gründe Ader dieses Verbindungskabel bleiben ungenutzt.

Die jeweils zweiten Pins am Eingang des Step-Down-Konverters nutzen wir, um die Spannung der Batteriebox unverändert zum Motortreiber auf dem Calliope zu führen. Hierzu verbinden wir diese mit VM bzw GND auf dem Calliope.

Aus dem USB-Ausgang des Step-Down-Converter kann der Calliope mit Strom versorgt werden (5 Volt). Hierzu nutzen wir das kurze USB-Kabel aus dem Lieferumfang des Calliope mini.


## Programmierung

Wir programmieren das Fahrzeug mit dem Editor [Makecode](https://makecode.calliope.cc/). Die Befehle für die Motoren befinden sich unter "Motoren".

Zunächst machen wir einen einfachen Test, bei dem das Fahrzeug nach Druck auf Button A zwei Sekunden gerade aus, dann zwei Sekunden rechts, dann zwei Sekunden links und dann nochmals zwei Sekunden geradeaus fährt

![Makecode Zwei-Motor-Betrieb](/images/makecode_dual_motor_test.png) 


Hier der Javascript-Code für Copy/Paste:
```javascript
input.onButtonPressed(Button.A, () => {
    motors.dualMotorPower(Motor.A, 100)
    motors.dualMotorPower(Motor.B, 100)
    basic.pause(2000)
    motors.dualMotorPower(Motor.B, 0)
    basic.pause(2000)
    motors.dualMotorPower(Motor.B, 100)
    motors.dualMotorPower(Motor.A, 0)
    basic.pause(2000)
    motors.dualMotorPower(Motor.A, 100)
    basic.pause(2000)
    motors.dualMotorPower(Motor.A, 0)
    motors.dualMotorPower(Motor.B, 0)
})
```

Aus Gründen der Übersichtlichkeit und Wiederverwendbarkeit definieren wir vier Funktionen für die Fahrzeugsteuerung. Hier wird zusätzlich die Richtung auf der LED-Matrix des Calliopes angezeigt. Wenn die angezeigte Richtung nicht stimmt, kann man dies dadurch korrigieren, dass man die Symbole ändert oder indem man die Motoranschlüsse vertauscht (gelb und grün) bzw. (links und rechts)

```javascript
function fahre_geradeaus () {
    motors.dualMotorPower(Motor.A, 100)
    motors.dualMotorPower(Motor.B, 100)
    basic.showLeds(`
        . . # . .
        . . # . .
        # . # . #
        . # # # .
        . . # . .
        `)
}
function fahre_rechts () {
    motors.dualMotorPower(Motor.A, 100)
    motors.dualMotorPower(Motor.B, 0)
    basic.showLeds(`
        . . # . .
        . . . # .
        # # # # #
        . . . # .
        . . # . .
        `)
}
function fahre_links () {
    motors.dualMotorPower(Motor.A, 0)
    motors.dualMotorPower(Motor.B, 100)
    basic.showLeds(`
        . . # . .
        . # . . .
        # # # # #
        . # . . .
        . . # . .
        `)
}
function stoppe () {
    motors.dualMotorPower(Motor.A, 0)
    motors.dualMotorPower(Motor.B, 0)
    basic.showLeds(`
        . . . . .
        . . . . .
        . . . . .
        . . . . .
        . . . . .
        `)
}
input.onButtonPressed(Button.A, function () {
    fahre_geradeaus()
    basic.pause(2000)
    fahre_links()
    basic.pause(2000)
    fahre_rechts()
    basic.pause(2000)
    fahre_geradeaus()
    basic.pause(2000)
    stoppe()
})
```

## Geschwindigkeit ändern

Im obigen Beispiel wird die Geschwindigkeit der Motoren immer auf 100% gesetzt. Durch Reduzierung dieses Wertes kann die Geschwindigkeit reduziert werden.

Zu beachten ist, dass sich die Motoren erst ab einem bestimmten Wert drehen, und dass erst ab einem noch höheren Wert das Fahrzeug aus dem Stand heraus gestartet werden kann.

Welche Werte das sind, hängt vom verwendeten Motor (Drehmoment, Drehzahl), von ggf. verwendeten Zahnrädernm, von den verwendeten Rädern und auch von den Eigenschaften des Untergrunds ab.

Man kann die Werte durch Tests ermitteln, indem man die Geschwindigkeit in Stufen von 0 bis 100 erhöht, bzw. reduziert.

((Programm))



## Genauer steuern

Wir haben nun zwar die Mögichkeit, das Fahrzeug vorwärts zu bewegen und zu drehen, die Bewegungen lassen sich jedoch nicht genau kontrollieren. Befehle wie "Fahre genau 1 Meter weit" oder "Drehe um 90° nach rechts" sind bislang noch nicht möglich.

Hierfür könneten die erforderlichen Werte mit initialen Messungen ermittelt werden:

Wie weit fährt das Fahrzeug in x Sekunden?
Um wieviel Grad dreht das Fahrzeug in x Sekunden?

((Programm))

((Ergebnisse))

((verbesserte Funktionen))

Nachteil dieser Vorgehensweise ist, dass sich die Motoren bei sinkender Batteriespannung langsamer drehen und damit die initiale Messung nicht mehr gültig ist. 

Neben diesen einmaligen manuellen Messungen gibt es aber auch die Möglichkeit, die Bewegungen durch Sensoren zu messen und diese im Programm auszuwerten:

Mit einem Encoder können die Umdrehungen des Motors gezählt und damit die Geschwindigkeit berechnet werden, siehe
https://www.youtube.com/watch?v=cLtMcqRetO0
https://www.youtube.com/watch?v=oLBYHbLO8W0

Ein Beschleunigungs- und Lagesensor bzw. Kompass kann verwendet werden um Drehungen oder sonstige Bewegungen im Raum auszuwerten. Ein derartiger Sensor ist im Calliope enthalten.
