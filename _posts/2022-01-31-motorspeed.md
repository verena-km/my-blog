---
layout: post
title:  "Umdrehungen zählen mit dem Speedsensor"
date:   2022-01-31
tags: Sensor Calliope Raspberry 
---

* TOC
{:toc}

Beim Einsatz vom Gleichstrommotoren hängt die Geschwindigkeit von der aktuellen Spannung der Batterien bzw. Akkus ab. Daher ist es schwierig, festgelegte Distanzen (z.B. 50 cm vorwärts) zu fahren. Zudem drehen sich selbst baugleiche Motoren nicht immer mit der gleichen Geschwindigkeit.

Was hilft, sind Sensoren, die die Anzahl der Umdrehungen ermitteln, beispielsweise der "Speedsensor LM393 mit Lochscheibe". Die mitgelieferte Loschscheibe passt jedoch nicht auf Lego-Achsen.

![Bild Sensor mit Lochscheibe](/images/foto_speedsensor_scheibe.jpg)

Eine Anleitung findet man hier: 
[https://joy-it.net/files/files/Produkte/SEN-Speed/SEN-Speed-Anleitung-20201015.pdf](https://joy-it.net/files/files/Produkte/SEN-Speed/SEN-Speed-Anleitung-20201015.pdf)

Der Speedsensor hat vier Anschlüsse, wovon aber nur drei benötigt werden:

* VCC - Spannungsversorgung 3,3 bis 5 V
* GND - Ground
* D0 - digitales Signal für den Zustand der Schranke

Der Sensor gibt auf seinem Pin D0 eine 1 aus, wenn sich etwas in der Lichtschranke befindet und eine 0, wenn sich nichts in der Lichtschranke befindet. Platziert man die Lochscheibe in der Lichtschranke und dreht diese (manuell oder durch einen Motor) kann man durch Zählen der Wechsel die Anzahl der Umdrehungen bestimmen.

Zur Verwendung mit LEGO-Motoren benötigen wir eine passende Lochscheibe. Man kann z.B. die LEGO-Technic-Riemenscheibe (4185) mit 6 Löchern verwenden Bei einer Umdrehung misst man also 6 Wechsel von "Offen auf Zu" und 6 Wechsel von "Zu  auf Offen". Man kann also die Umdrehungszahl auf 1/12 Umdrehung genau bestimmen.

Den Speedsensor LM393 habe ich auf einen dünnen Lego Technic Liftarm 1x3 (6632) mit Achslöchern geklebt und zwar so, dass ein Achsloch frei bleibt. Durch dieses kann man dann den Sensor mit Hilfe eine Achse in passendem Abstand um die Lochscheibe herum platzieren.

![Riemenscheibe als Lochscheibe](/images/foto_speedsensor_riemenscheibe.jpg) 


## Funktionstest am Calliope

Für einen Testaufbau ohne Geschwindigkeitsveränderung des Motors verwenden wir:

* die Lego-Batteriebox
* einen Lego-Motor
* den Speedsensor LM393
* ein Lego-Technic-Riemenscheibe als Lochscheibe
* den Calliope mit den herausgeführten Pin-Leisten

![Minimaler Testaufbau Speedsensor](/images/foto_testaufbau_speedsensor_1.jpg) 

Wir stecken die Riemenscheibe auf eine mit dem Motor verbundene Lego-Achse. Der Motor erhält sein Strom in diesem einfachen Aufbau direkt aus der Batteriebox, der Calliope erhält den Strom über USB.

Die Riemenscheibe platzieren wir in der Lichtschranke des Sensors.

Der Sensor hat vier Ausgänge. Diese verbinden wir wie folgt:
* Pin VCC am Sensor mit VCC am Calliope
* Pin GND am Sensor mit GND
* Pin D0 am Sensor mit C1 am Calliope

![Anschluss Speedsensor an Calliope](/images/fritzing_calliope_anschluss_speedsensor.png)

Startet man nun den Motor durch Drücken des Schalters an der Batteriebox, so dreht sich die Lochscheibe innerhalb der Lichtschranke. 

Nachfolgendes Programm zählt die Wechsel zwischen 0 und 1 sowie 1 und 0 und berechnet daraus die Umdrehungen. Die Werte werden für den Test auf der seriellen Schnittstelle ausgegeben.

![Makecode Speedsensor](/images/makecode_speed_sensor1.png)

```javascript
let anzahl_umdrehungen = 0
let anzahl_wechsel = 0
let alter_wert = 0
let neuer_wert = 0
pins.setPull(DigitalPin.P1, PinPullMode.PullUp)
basic.forever(function () {
    neuer_wert = pins.digitalReadPin(DigitalPin.P1)
    serial.writeValue("neuer_wert", neuer_wert)
    if (neuer_wert != alter_wert) {
        alter_wert = neuer_wert
        anzahl_wechsel += 1
        anzahl_umdrehungen = anzahl_wechsel / 12
    }
    serial.writeValue("anzahl_wechsel", anzahl_wechsel)
    serial.writeValue("anzahl_umdrehungen", anzahl_umdrehungen)
})
```

Damit die vom Sensor erzeugten Spannungen vom Calliope richtig als 1 bzw. 0 erkannt werden, muss der Pin als PullUp konfiguriert werden (in Makecode etwas merkwürdig übersetzt als "setzte Anziehungskraft").

Danach werden in einer Schleife dauerhaft die Sensorwerte und in der Variable "neuer_wert" ermittelt. Wenn der neue Wert vom alten abweicht, wird die Variable alter_wert auf den neuen Wert geändert. Zudem wird die Variable anzahl_wechsel um 1 erhöht und die Variable anzahl_umdrehungen neu berechnet.

Die Werte werden bei jedem Schleifendurchgang über die serielle Schnittstelle ausgegeben.

Man kann aber auch den onPulse-Event-Handler verwenden. Dabei wird bei jedem Wechsel von 0 auf 1 sowie bei jedem Wechsel von 1 auf 0 ein Event ausgelöst, für das man jeweils entsprechende Behandlungsroutinen erstellen kann. Diese werden dann durchgeführt, sobald das entsprechende Event auftritt. 


```javascript
let anzahl_umdrehungen = 0
let anzahl_wechsel = 0
pins.onPulsed(DigitalPin.P1, PulseValue.High, () => {
    serial.writeValue("high", pins.pulseDuration())
    anzahl_wechsel += 1
    anzahl_umdrehungen = anzahl_wechsel / 12
    serial.writeValue("anzahl_wechsel", anzahl_wechsel)
    serial.writeValue("anzahl_umdrehungen", anzahl_umdrehungen)
})
pins.onPulsed(DigitalPin.P1, PulseValue.Low, () => {
    serial.writeValue("low", pins.pulseDuration())
    anzahl_wechsel += 1
    anzahl_umdrehungen = anzahl_wechsel / 12
    serial.writeValue("anzahl_wechsel", anzahl_wechsel)
    serial.writeValue("anzahl_umdrehungen", anzahl_umdrehungen)
})
pins.setPull(DigitalPin.P1, PinPullMode.PullUp)
```

Diese Vorgehensweise hat mehrere Vorteile:
* Man spart sich das laufende Prüfen, ob sich der an Pin1 anliegende Wert geändert hat.
* Man erhält direkt auch die Zeitdauer des Pulses (pulseDuration), aus der man dann auch leicht die Dauer einer Umdrehung berechnen kann. 

Achtung: Beim Test wurden bei einem schnellen Wechsel zwischen 0 und 1 nicht alle Ausgaben an die Serielle Schnittstelle geschrieben.

## Funktionstest am Raspi

Einen einfachen Funktionstest kann man auch mit dem Raspi machen. Man verbindet

* Pin VCC am Sensor mit Pin 3,3 V am Raspi
* Pin GND am Sensor mit Pin Ground am Raspi
* Pin D0 am Sensor mit Pin GPIO 27 am Raspi

Mit folgendem Programm kann man die Lichtschranke testen, beispielsweise indem man einen Karton in die Schranke hält.

```python 
from gpiozero import DigitalInputDevice
from signal import pause

def activated():
    print("Offen")

def deactivated():
    print("Geschlossen")    

speedsensor = DigitalInputDevice(27)

speedsensor.when_activated = activated
speedsensor.when_deactivated = deactivated

pause()
```

Wir verwenden hier bei die Klassse DigitalInputDevice aus der gpiozero-Bibliothek und definieren zwei Funktionen, die ausgeführt werden, wenn die Schranke geschlossen bzw. die Schranke geöffnet wird.

## Geschwindigkeitsmessung am Raspi

Um die Geschwindigkeit zu messen, platzieren wir die Riemenscheibe als Lochscheibe in die Lichtschranke und treiben diese wieder mit dem Lego-Motor an, der aus der Lego-Batteriebox mit Strom versorgt wird.

```python 
from gpiozero import DigitalInputDevice
from time import sleep

# Methode die zaehlt wie oft die Lichtschranke ausloest
def count(self):
    global counter
    counter = counter + 1

counter = 0
interval = 1 # sekunde
loecher = 6 

speedsensor = DigitalInputDevice(27)
speedsensor.when_activated = count
speedsensor.when_deactivated = count

while True:
    print("Umdrehungen pro Sekunde:" + str(round(counter/(loecher*2),1)))
    counter = 0
    sleep(interval)
```
Hier werden sowohl das Öffnen als auch das Schließen der Schranke gezählt. Jede Sekunde werden die gezählten Umdrehungen ausgeben und der Zähler anschließen zurückgesetzt.

Mit diesem Verfahren kann man die Drehzahl von Motoren ermitteln und mit anderen Motoren vergleichen.

## Messung zur Steuerung verwenden

Man kann natürlich auch die gemessenen Werte verwenden, um den Motor zu steuern, also beispielsweise einen Motor nur eine zuvor festgelegte Anzahl von Umdrehungen durchführen lassen oder ein Fahrzeug damit eine bestimmte Strecke vorwärts fahren lassen. 
