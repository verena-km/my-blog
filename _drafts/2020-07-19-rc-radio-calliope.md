---
layout: post
title:  "Calliope steuert Calliope"
date:   2020-07-19 17:08:49 +0200
tags: calliope fernsteuerung funk
---
Der Prototyp mit Calliope soll nun mit einem anderen Calliope ferngesteuert werden.
Hierfür nutzen wir das Funk-Modul, mit dem aus mehreren Calliopes (und auch MicroBits) eine Funkgruppe gebildet werden kann. Jeder in der Funkgruppe kann dann Daten (Text oder Zahlen) senden und empfangen.

Die Befehle findet man in Makecode unter "Funk", einige Befehle dann auch unter "Mehr".

## Komponenten und Aufbau

Für den Robby-Prototyp haben nutzen wir zwei Calliope. Einer verbaut auf dem Fahrzeug als Empfänger für die Befehle, der weitere ist die Fernbedienung.

Komponenten und Aufbau des Fahrzeugs sind gleich wie im Beitrag [Erster Prototyp mit Calliope]({% post_url 2020-07-19-modell-a-calliope %})


## Variante 1: Nutzung der Buttons

In der ersten Variante der Fernsteuerung soll das Fahrzeug
* beim Drücken von A und B geradeaus fahren
* beim Drücken von A links fahren
* beim Drücken von B rechts fahren
* beim Schütteln anhalten

Die Calliope-Fernbedienung nutzt folgendes Programm. Es übermittelt abhängig von der Eingabe Zahlencodes von 0 bis 2 über Funk an die Mitglieder der Funkgruppe 0:


```javascript
input.onGesture(Gesture.Shake, () => {
    radio.sendNumber(0)
})
input.onButtonPressed(Button.AB, () => {
    radio.sendNumber(1)
})
input.onButtonPressed(Button.A, () => {
    radio.sendNumber(2)
})
input.onButtonPressed(Button.B, () => {
    radio.sendNumber(3)
})
radio.setGroup(0)
```

Der auf dem Fahrzeug verbaute Calliope nutzt folgendes Programm. Es reagiert auf die über Funk empfangene Daten und führt die entsprechenden Befehle zur Motorsteuerung aus.

## Variante 2: Nutzung des Lagesensors

Der Calliope hat einen kombinierten Beschleunigungs- und Lagesensor.

Mit Makecode kann man von ihm folgende Werte erhalten:
* Beschleunigung(mg) x - input.acceleration(Dimension.X)
* Beschleunigung(mg) y - input.acceleration(Dimension.Y)
* Beschleunigung(mg) z - input.acceleration(Dimension.Z)
* Beschleunigung(mg) Staerke - input.acceleration(Dimension.Strength)
* Rotation(°) Winkel - input.rotation(Rotation.Pitch))
* Rotation(°) rollen - input.rotation(Rotation.Roll))

Mit einem Testprogramm lassen wir die Werte auf der seriellen Schnittstelle ausgeben.

![Makecode Beschleunigungs- und Rotationswerte](/images/makecode_test_acc_gyro.png) 

```javascript
basic.forever(() => {
    serial.writeString("Beschleunigung: ")
    serial.writeNumber(input.acceleration(Dimension.X))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Y))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Z))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Strength))
    serial.writeString(" Rotation: ")
    serial.writeNumber(input.rotation(Rotation.Pitch))
    serial.writeString(" ")
    serial.writeNumber(input.rotation(Rotation.Roll))
    serial.writeLine("")
    basic.pause(500)
})
```

Was sind das nun für Werte? Fangen wir mal mit der Rotation an.

Der [Wikipedia-Artikel zu Roll-Nick-Gier-Winkel](https://de.wikipedia.org/wiki/Roll-Nick-Gier-Winkel ) zeigt anschaulich die Drehbewegungen um die drei Achsen. Übertragen auf den Calliope kann man desseb USB-Schnittstelle als "Spitze" des Fahrzeugs / Flugzeugs sehen.

Mit dem Rotations-Befehl des Calliope erhalten wir Werte zum "Winkel/Nicken (Pitch)" und zum "Rollen (Roll)". 

Der Wert "Winkel/Nicken(Pitch)" verändert sich, wenn man die Spitze nach oben bzw. unten dreht:

Bei Makecode:
* liegend (LED nach oben): pitch = 0°
* stehend (USB nach oben): pitch = 90°
* liegend (LED nach unten) pitch = 0°
* auf dem Kopf (USB nach unten) pitch = -90°
Bei Makecode - neue beta:
* liegend (LED nach oben): pitch = 0°
* stehend (USB nach oben): pitch = 90°
* liegend (LED nach unten) pitch =  180°/-180°
* auf dem Kopf (USB nach unten) pitch = -90°

Der Wert "Rollen (Roll)" verändert sich, wenn man den Callipe nach links bzw. nach rechts neigt:
Bei Makecode:
* liegend (LED nach oben): roll = 180°/-180°
* nach links (PIN 0 unten): roll = -90°
* liegend (LED nach unten): roll = 0°
* nach rechts (PIN 3 unten): roll = 90°

Bei Makecode - neue Beta
* liegend (LED nach oben): roll = 0°
* nach links (PIN 0 unten): roll = -90°
* liegend (LED nach unten): roll = 180°/-180°
* nach rechts (PIN 3 unten): roll = 90°

Bei den Beschleunigungswerten wird die Beschleunigung entlang der Achsen angegeben. Dabei verläuft
* die x-Achse von links nach rechts, also auf einer gedachten Verbindung von Pin 0 und Pin3
* die y-Achse von vorne nach hinten, also von der USB-Buchse nach unten
* die z-Achse vertikal zum Calliope

Die Beschleunigung wird in milli-g, also in Tausendstel der Erdanziehung gemessen. In den gemessenen Werten ist die Erdanziehung enthalten.

Ein flach liegender und nicht bewegter Calliope erzeugt die Werte x=0, y=0, z=-1000. Liegt er mit den LEDs nach unten sind die Werte x=0, y=0, z=1000. Mit dem Beschleunigungssensor sieht man im Ruhezustand (durch die Erdanziehung) ob der Calliope geneigt ist (Roll oder Pitch). Einen Roll sieht man am sich ändernden x-Wert, einen Pitch am sich ändernden y-Wert.

Ist der Calliope nicht geneigt, geben x und y die lineare Beschleunigung wieder.

Für die Fernbedienung nutzen wir den Lagesensor. Bei einer Neigung nach links soll das Fahrzeug nach links, bei einer Neigung nach rechts soll das Fahrzeug nach rechts fahren. Bei einer Neigung nach vorne soll der Calliope schneller, bei einer Neigung nach hinten langsamer fahren.

Um diesen Effekt zu erzielen, schauen wir uns die Wertebereiche an (Makecode beta)
Als maximale Neigungwinkel nach links und rechts nehmen wir jeweils 90°, damit ergeben sich folgende Werte für Roll:

Linksfahren:    Roll von 0° geradeaus bis -90° (stark links)
Rechtsfahren:   Roll von 0° geradeaus bis 90° (stark rechts)

Die Geschwindigkeit soll 0 sein wenn der Calliope "steht" und maximal wenn er liegt. Es sollen also folgende Werte für Pitch vorliegen. 

Pitch von 90° = 0% Geschwindigkeit, Pitch von 0° = 100% Geschwindigkeit

((Zudem sollte der Calliope mit den LEDs nach oben gehalten werden, was mit einem Beschleunigungswert von z > 0 sichergestellt werden kann.))

Vor der Übertragung bilden wir die Roll-Werte für links und rechts auf eine verständlichere Skala ab:
stark links = -100
geradeaus = 0
stark rechts = 100

Gleiches machen wir für die Geschwindigkeit:
0% = 0
100% = 100

let richtung = 0
let geschwindigkeit = 0
radio.setGroup(10)
basic.forever(function () {
    serial.writeString("Beschleunigung: ")
    serial.writeNumber(input.acceleration(Dimension.X))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Y))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Z))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Strength))
    serial.writeString(" Rotation: ")
    serial.writeNumber(input.rotation(Rotation.Pitch))
    serial.writeString(" ")
    serial.writeNumber(input.rotation(Rotation.Roll))
    serial.writeLine("")
    if (input.rotation(Rotation.Pitch) < 0) {
        geschwindigkeit = 100
    }
    if (input.rotation(Rotation.Pitch) > 90) {
        geschwindigkeit = 0
    }
    if (input.rotation(Rotation.Pitch) >= 0 && input.rotation(Rotation.Pitch) <= 90) {
        geschwindigkeit = pins.map(
        input.rotation(Rotation.Pitch),
        90,
        0,
        0,
        100
        )
    }
    if (input.rotation(Rotation.Roll) >= -90 && input.rotation(Rotation.Roll) <= 90) {
        richtung = pins.map(
        input.rotation(Rotation.Roll),
        -90,
        90,
        -100,
        100
        )
    }
    serial.writeValue("richtung", Math.round(richtung))
    serial.writeValue("geschwindigkeit", Math.round(geschwindigkeit))
    radio.sendValue("richtung", Math.round(richtung))
    radio.sendValue("geschwindigkeit", Math.round(geschwindigkeit))
    basic.pause(1000)
})



let richtung = 0
let geschwindigkeit = 0
radio.setGroup(10)
basic.forever(function () {
    serial.writeString("Beschleunigung: ")
    serial.writeNumber(input.acceleration(Dimension.X))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Y))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Z))
    serial.writeString(" ")
    serial.writeNumber(input.acceleration(Dimension.Strength))
    serial.writeString(" Rotation: ")
    serial.writeNumber(input.rotation(Rotation.Pitch))
    serial.writeString(" ")
    serial.writeNumber(input.rotation(Rotation.Roll))
    serial.writeLine("")
    if (input.rotation(Rotation.Pitch) < 0) {
        geschwindigkeit = 100
    }
    if (input.rotation(Rotation.Pitch) > 90) {
        geschwindigkeit = 0
    }
    if (input.rotation(Rotation.Pitch) >= 0 && input.rotation(Rotation.Pitch) <= 90) {
        geschwindigkeit = pins.map(
        input.rotation(Rotation.Pitch),
        90,
        0,
        0,
        100
        )
    }
    if (input.rotation(Rotation.Roll) >= -90 && input.rotation(Rotation.Roll) <= 90) {
        richtung = pins.map(
        input.rotation(Rotation.Roll),
        -90,
        90,
        -100,
        100
        )
    }
    serial.writeValue("richtung", Math.round(richtung))
    serial.writeValue("geschwindigkeit", Math.round(geschwindigkeit))
    radio.sendValue("richtung", Math.round(richtung))
    radio.sendValue("geschwindigkeit", Math.round(geschwindigkeit))
    basic.pause(200)
})


radio.onReceivedValue(function (name, value) {
    serial.writeValue(name, value)
    if (name == "richtung") {
        richtung = value
    }
    if (name == "geschwin") {
        geschwindigkeit = value
    }
    serial.writeValue("geschwindigkeit", geschwindigkeit)
    if (richtung < 0) {
        motors.dualMotorPower(Motor.A, geschwindigkeit - Math.abs(richtung))
        motors.dualMotorPower(Motor.B, geschwindigkeit)
    } else {
        motors.dualMotorPower(Motor.A, geschwindigkeit)
        motors.dualMotorPower(Motor.B, geschwindigkeit - Math.abs(richtung))
    }
})
let geschwindigkeit = 0
let richtung = 0
radio.setGroup(10)
motors.dualMotorPower(Motor.A, 0)
motors.dualMotorPower(Motor.B, 0)
