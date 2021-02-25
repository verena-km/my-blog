---
layout: post
title:  "Digitale am Calliope"
date:   2021-01-03
tags: GPIO Pullup Pulldown Calliope Events
---

## Konfigurationsmöglichkeiten der digitalen Einggänge

Beim Calliope hat für jeden Eingangs-Pin folgende Konfigurationsmöglichkeiten:
* den PinPullMode
* den PinEventType

Mit dem PinPullMode stellt man ein, ob für den Pin softwareseitig ein PullUp- oder ein PullDown- oder kein Widerstand zugeschaltet werden soll.

```javascript
pins.setPull(DigitalPin.P0, PinPullMode.PullNone)
pins.setPull(DigitalPin.P0, PinPullMode.PullUp)
pins.setPull(DigitalPin.P0, PinPullMode.PullDown)
```
Mit dem PinEventType kann man einstellen, welche Events für den Pin erzeugt werden. Dabei gibt es folgende Werte:
* Rand-Ereignisse (PinEventType.Edge)
* Puls-Ereignisse (PinEventType.Pulse)
* Berührungs-Ereignisse (PinEventType.Touch)
* keine Ereignisse (PinEventType.None)

Gesetzt wird der PinEventType wie folgt:

```javascript
pins.setEvents(DigitalPin.P0, PinEventType.Touch)
```
Eigentlich muss man den PinEventType nicht setzen, dass passiert wohl durch die Nutzung der einzelnen Events automatisch.

## Auslesen der Werte an einem digitalen Eingang

Werte an einem digitalen Eingang können mit 'digitalReadPin' ausgelesen werden:

```javascript
basic.forever(function () {
    serial.writeNumber(pins.digitalReadPin(DigitalPin.P0))
    basic.pause(500)
})
```
Die Funktion liefert 1, wenn zwischen dem Pin und dem Pluspol eine Verbindung besteht und 0, wenn nicht.

Will man feststellen, ob zwischen dem Pin und dem Minuspol eine Verbindung besteht, muss man den 'PinPullMode' auf 'PinPullMode.PullUp' setzen:

```javascript
pins.setPull(DigitalPin.P0, PinPullMode.PullUp)
basic.forever(function () {
    serial.writeNumber(pins.digitalReadPin(DigitalPin.P0))
    basic.pause(500)
})
```
Die Funktion liefert nun eine 0, wenn zwischen dem Pin und dem Minuspol eine Verbindung besteht und eine 1, wenn nicht.

## Reaktion auf Ereignisse
Statt regelmäßig die Werte abzufragen, kann man auch Eventhandler verwenden um über bestimmte Ereignisse informiert zu werden. 

Beim Calliope (Makecode) gibt es folgende vorgefertigten Eventhandler für digitale Eingänge:
* pins.onPulsed
* input.onPinPressed
* input.onPinReleased


## onPinPressed und onPinReleased
* werden jeweils beim Öffnen einer Verbindung zwischen Pin und GND ausgeführt
* onPinPressed wird ausgeführt, wenn der Pin gedrückt und innerhalb einer Sekunde wieder losgelassen wird
* onPinReleased wird immer ausgeführt, wenn der Pin losgelassen wird
* Konfiguriert man beide, wird nur onPinReleased ausgeführt!
* erfordert PinEventType.Touch, PinEventType.Pulse oder keine explizite Angabe eines PinEventTypes
* wecher PinPullMode ist egal

```javascript
input.onPinPressed(TouchPin.P0, function () {
    serial.writeLine("pressed")
})
input.onPinReleased(TouchPin.P0, function () {
    serial.writeLine("released")
})
```

## onPulsed

Bei onPulsed kann man angeben, ob man Event beim Wechsel von High nach Low oder von Low nach High informiert werden will. 

pins.onPulsed(DigitalPin.P0, PulseValue.High, function () {
    serial.writeValue("high", pins.pulseDuration())
})
pins.onPulsed(DigitalPin.P0, PulseValue.Low, function () {
    serial.writeValue("low", pins.pulseDuration())
})

Beim PinPullMode.PullDown gilt folgendes:
* es werden die Wechsel zwischen Pin und VCC ausgewerttet

* Bei PulseValue.High wird die Funktion jeweils ausgeführt beim Trennen einer Verbindung zwischen Pin und VCC. Dabei entspricht die Impulsdauer dem Zeitraum in dem die Verbindung bestand - also so lange es High war.

* Bei PulseValue.Low wird die Funktion jeweils ausgeführt beim Herstellen einer Verbindung zwischen Pin und VCC. Dabei entspricht die Impulsdauer dem Zeitraum, in dem keine Verbindung bestand - also wie lange es Low war.

Beim PinPullMode.PullUp gilt folgendes:

* bei PinPullMode.PullUp werden die Wechsel zwischen Pin und Ground ausgewertet

* Bei PulseValue.High wird die Funktion jeweils ausgeführt beim Herstellen einer Verbindung zwischen Pin und Ground. Dabei entspricht die Impulsdauer dem Zeitraum in dem keine Verbindung bestand - also so lange es High war.

* Bei PulseValue.Low wird die Funktion jeweils ausgeführt beim Trennen einer Verbindung zwischen Pin und Ground. Dabei entspricht die Impulsdauer dem Zeitraum, in dem eine Verbindung bestand - also wie lange es Low war.

Mit Hilfe beider onPulsed-EventHandler kann man somit Aktiviäten beim Wechsel von 0 nach 1 als auch von 1 nach 0 ausführen lassen und auch die Zeitdauer des jeweiligen Pulses nutzen.


