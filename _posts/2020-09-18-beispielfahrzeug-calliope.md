---
layout: post
title:  "Erster Prototyp mit Calliope"
date:   2020-09-18
tags: Calliope Beispielfahrzeug
---

![Beispielfahrzeug zwei Antriebsräder und Stützrad mit Callipe](/images/foto_modell_a_calliope.jpg)

Für den ersten Prototyp nehme ich den Calliope und dessen integrierten Motortreiber. Das Fahrzeug hat zwei angetriebene Räder und ein Stützrad. Aufgrund der Einschränkungen des Calliope-Motortreibers gibt es leider keinen Rückwärtsgang.

## Komponenten und Aufbau

Ich nutze folgende Komponenten:
* 1 Calliope Mini mit herausgeführter Pinleiste
* 2 Lego M-Motoren
* 1 Lego Power-Functions-Batteriebox
* 1 Step-Down-Converter

Folgender Schaltplan zeigt die Verkabelung:
![Schaltplan Zwei-Motor-Betrieb](/images/fritzing_calliope_dual_motor_stepdown.png)

Die Lego-Batteriebox wird mit den Eingängen des Step-Down-Konverters verbunden. Hierfür verwenden wir ein modifiziertes Lego-Verlängerungskabel und stecken den Lego-Stecker auf die Batteriebox. Am Eingang des Step-Down-Konverters schrauben wir jeweils bei "+" und bei "minus" zwei Jumper-Kabel-Verbindungstecker ein. Dananch verbindet man die rote Ader des Verbindungskabel mit einem Pin des mit "+" gekennzeichneten Eingangs, die schwarze Ader mit einem Pin des mit "-" gekennzeichneten Eingangs des Step-Down-Konverters. Das gelbe und gründe Ader dieses Verbindungskabel bleiben ungenutzt.

Die jeweils zweiten Pins am Eingang des Step-Down-Konverters nutzen wir, um die Spannung der Batteriebox unverändert zum Motortreiber auf dem Calliope zu führen. Hierzu verbinden wir diese mit VM bzw GND auf dem Calliope.

Die Lego-Motoren verbinden wir jeweils mit einem modifizierten Lego-Verlägerungskabel und nutzen von diesen jeweils die gelbe und die grüne Ader. Diese verbinden wir mit dem Motortreiber des Calliope Mini.

Aus dem USB-Ausgang des Step-Down-Converter kann der Calliope mit Strom versorgt werden (5 Volt). Hierzu nutzen wir das kurze USB-Kabel aus dem Lieferumfang des Calliope mini. Zur Übertragung von Programmen vom Computer auf den Calliope nutze ich ein weiteres, relativ langes USB-Kabel und stecke dann für den mobilen Einsatz um.

## Programmierung

Wir programmieren das Fahrzeug mit dem Editor [Makecode](https://makecode.calliope.cc/). Die Befehle für die Motoren befinden sich unter "Motoren".

### Erstes Testprogramm

Zunächst machen wir einen einfachen Test, bei dem das Fahrzeug nach Druck auf Button A zwei Sekunden gerade aus, dann zwei Sekunden links, dann zwei Sekunden rechts und dann nochmals zwei Sekunden geradeaus fährt.

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

Anmerkung: Künftig werde ich hier nur noch den Javascript-Code zeigen. Wenn man diesen nach Makecode in die Ansicht "Javascript" kopiert kann man sich anschließend die grafische Darstellung anschauen, indem man zu "Blöcke" wechselt.

### Nutzung von Funktionen

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

### Geschwindigkeit ändern

Im obigen Beispiel wurde die Geschwindigkeit der Motoren immer auf 100% gesetzt. Durch Reduzierung des Prozentwerts kann die Geschwindigkeit reduziert werden.

Zu beachten ist, dass sich die Motoren erst ab einem bestimmten Wert drehen, und dass erst ab einem noch höheren Wert das Fahrzeug aus dem Stand heraus gestartet werden kann.

Welche Werte das sind, hängt vom verwendeten Motor (Drehmoment, Drehzahl), von ggf. verwendeten Zahnrädern, von den verwendeten Rädern und auch von den Eigenschaften des Untergrunds sowie von der am Motor anliegenden Spannung ab.

Man kann die Werte durch Tests ermitteln, indem man die Geschwindigkeit in Stufen von 0 bis 100 erhöht.

```javascript
input.onButtonPressed(Button.A, function () {
    motors.dualMotorPower(Motor.A, speed)
    motors.dualMotorPower(Motor.B, speed)
    while (speed < 100) {
        speed += 5
        motors.dualMotorPower(Motor.A, speed)
        motors.dualMotorPower(Motor.B, speed)
        basic.showNumber(speed)
        basic.pause(5000)
    }
    motors.dualMotorPower(Motor.A, 0)
    motors.dualMotorPower(Motor.B, 0)
})
let speed = 0
speed = 0
```

Aus dem Stand startet mein Robotorfahrzeug bei einem Wert von 55 %.


### Genauer steuern

Wir haben nun zwar die Mögichkeit, das Fahrzeug vorwärts zu bewegen und zu drehen, die Bewegungen lassen sich jedoch nicht genau kontrollieren. Befehle wie "Fahre genau 1 Meter weit" oder "Drehe um 90° nach rechts" sind bislang noch nicht möglich.

In einem einfachen Ansatz können die erforderlichen Werte mit initialen Messungen ermittelt werden:

**Wie weit fährt das Fahrzeug in x Sekunden?**

Hierfür kann man das Fahrzeug eine bestimmte Zeit (hier 5 Sekunden) fahren lassen und dann die Strecke nachmessen:

```javascript
input.onButtonPressed(Button.A, function () {
    motors.dualMotorPower(Motor.A, 100)
    motors.dualMotorPower(Motor.B, 100)
    basic.pause(5000)
    motors.dualMotorPower(Motor.A, 0)
    motors.dualMotorPower(Motor.B, 0)
})
```
Mein Fahrzeug hat in 5 Sekunden 60 cm zurücklegt, pro Sekunde also 12 cm. Oder: Für 10 cm benötigt es 0.833 Sekunden.

**Um wieviel Grad dreht das Fahrzeug in x Sekunden?**

Führt man die Drehung so aus, dass ein Motor an, der andere aber aus ist, bleibt das eine Rad stehen und das andere dreht sich um das stehende Rad. Das drehende Rad beschreibt also bei einer Drehung um 360° einen Kreis. Der Radius dieses Kreises ist der Abstand zwischen den Rädern. Den von dem drehenden Rad zurückgelegten Weg kann man daraus als Kreisumfang berechnen. Beträgt der Abstand der Räder 16,5 cm, so ist der Umfang 2 x π  x 16,5 cm, also 103,7 cm.

![Skizze Drehung Fahrzeug](/images/skizze_drehung_fahrzeug.png)

Legt man die obige Geschwindigkeit  von 0.833 Sekunden pro 10 cm zugrunde - dauert eine 360°-Drehung 8,6 Sekunden. Das stimmt aber nicht ganz, da wir ja beim Geradeausfahren zwei Motoren nutzen. Bei meinem Fahrzeug dauert eine 360°-Drehung z.B. 9,2 Sekunden.

```javascript
input.onButtonPressed(Button.A, function () {
    motors.dualMotorPower(Motor.A, 100)
    basic.pause(9200)
    motors.dualMotorPower(Motor.A, 0)
})
```
Die Drehgeschwindigkeit beträgt damit 39,1° pro Sekunde.


Man kann nun Geschwindigkeit und Drehgeschwindigkeit als Konstanten im Programm verwenden und Funktionen definieren, die das Fahrzeug eine als Parameter übergebende Strecke (in cm) fahren lassen bzw. es um eine bestimmte Gradzahl drehen lassen. Die Funktionen berechnen dann jeweils, wie lange die einzelnen Motoren auf 100% laufen müssen um den entsprechenden Weg zurückzulegen.

```javascript
let drehgeschwindigkeit = 0
let geschwindigkeit = 0
geschwindigkeit = 12
drehgeschwindigkeit = 39.1

function fahre_geradeaus (strecke: number) {
    motors.dualMotorPower(Motor.AB, 100)
    basic.pause(strecke / geschwindigkeit * 1000)
    motors.dualMotorPower(Motor.AB, 0)
}

function drehe_links (grad: number) {
    motors.dualMotorPower(Motor.A, 100)
    motors.dualMotorPower(Motor.B, 0)
    basic.pause(grad / drehgeschwindigkeit * 1000)
    motors.dualMotorPower(Motor.AB, 0)
}
function drehe_rechts (grad: number) {
    motors.dualMotorPower(Motor.A, 0)
    motors.dualMotorPower(Motor.B, 100)
    basic.pause(grad / drehgeschwindigkeit * 1000)
    motors.dualMotorPower(Motor.AB, 0)
}
input.onButtonPressed(Button.A, function () {
    fahre_geradeaus(20)
    drehe_links(90)
    fahre_geradeaus(20)
    drehe_links(90)
    fahre_geradeaus(20)
    drehe_links(90)
    fahre_geradeaus(20)
    drehe_links(90)
})

input.onButtonPressed(Button.B, function () {
    fahre_geradeaus(20)
    drehe_rechts(120)
    fahre_geradeaus(20)
    drehe_rechts(120)
    fahre_geradeaus(20)
    drehe_rechts(120)

})
```
Die Funktionen werden in diesem Beispielprogramm genutzt, um beim Drücken von A ein Quadrat und bei Drücken von B ein gleichseitiges Dreieck abzufahren.

Nachteil dieser Vorgehensweise ist, dass sich die Motoren bei sinkender Batteriespannung langsamer drehen und damit die initiale Messung nicht mehr gültig ist. 

Neben diesen einmaligen manuellen Messungen gibt es aber auch die Möglichkeit, die Bewegungen durch Sensoren zu messen und diese im Programm auszuwerten. Mit einem Encoder können die Umdrehungen des Motors gezählt und damit die Geschwindigkeit berechnet werden. Mit einem Kompass kann dessen Ausrichtung für die Messung von Drehungen ausgewertet werden. 

Den Einsatz von Encodern bzw. des im Calliope Mini integrierten Kompass erläutere ich in einem späteren Post.
