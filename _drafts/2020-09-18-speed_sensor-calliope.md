---
layout: post
title:  "Geschwindigkeitskontrolle für den Calliope"
date:   2020-09-18 07:08:49 +0200
tags: Speedsensor LM393 Lochscheibe Encoder Calliope
---

Beim Einsatz vom Gleichstrommotoren hängt die Geschwindigkeit von der aktuellen Spannung der Batterien bzw. Akkus ab. Daher ist es schwierig, das genaue Fahren von Distanzen - z.B. "Fahre 50 cm geradeaus" zu programmieren.

Was hilft, sind Sensoren, die die Anzahl der Umdrehungen ermitteln. Solche sog. Encoder können z.B. sein:
* [Hall-Sensoren](https://de.wikipedia.org/wiki/Hall-Sensor)
* [Lichtschranken](https://de.wikipedia.org/wiki/Lichtschranke)

Zu letzteren gehört beispielsweise der "Speedsensor LM393 mit Lochscheibe". Enthalten ist auch eine Lochscheibe mit einer Bohrung, die aber nicht auf die Lego-Achsen passt.

![Bild Sensor mit Lochscheibe](/images/foto_speedsensor_scheibe.jpg) 

Zur Verwendung mit LEGO-Motoren benötigen wir also eine angepasste Lochscheibe. Für erste Experimente nutze ich eine LEGO-Technic-Riemenscheibe. Diese hat 6 Löcher. Bei einer Umdrehung misst man also 6 Wechsel von "Hell auf Dunkel" und 6 Wechsel von "Dunkel auf Hell". Man kann also die Umdrehungszahl auf 1/12 Umdrehung genau bestimmen.

![Riemenscheibe als Lochscheibe](/images/foto_motor_lochscheibe.jpg) 

## Testaufbau Speedsensor ohne Geschwindigkeitsveränderung

Für den ersten minimalen Testaufbau ohne Geschwindigkeitsveränderung der Motoren verwenden wir:

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
* Pin A0 am Sensor bleibt ungenutzt

![Anschluss Speedsensor an Calliope](/images/fritzing_calliope_anschluss_speedsensor.png)

Startet man nun den Motor durch Drücken des Schalters an der Batteriebox, so dreht sich die Lochscheibe innerhalb der Lichtschranke. Der Sensor gibt auf seinem Pin D0 eine 1 aus, wenn sich etwas in der Lichtschranke befindet und eine 0, wenn sich nichts in der Lichtschranke befindet.

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


## Testaufbau Speedsensor mit Geschwindigkeitsveränderung

In einem weiteren Testaufbau wollen wir nun den Motor über den Calliope ansteuern und so auch dessen Geschwindigkeit ändern.

Wir verwenden:
* die Lego-Batteriebox
* einen Lego-Motor
* den Speedsensor LM393
* ein Lego-Technic-Riemenscheibe als Lochscheibe
* den Callipe mit den herausgeführten Pin-Leisten
* zwei modifizierte Lego-Verbindungskabel


TODO


