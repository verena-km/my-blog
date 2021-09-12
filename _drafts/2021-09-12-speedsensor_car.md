---
layout: post
title:  "Speedsensor zur Motor- und Fahrzeugsteuerung"
date:   2021-09-12
tags: Speedsensor Robot Raspberry Beispielfahrzeug
---

Bei diesem Fahrzeug wird jedes der beiden Räder von einem Motor angetrieben. An jedem der beiden Motoren ist ein Speedsensor LM393 angebracht.

((TODO: Foto gesamt))

## Komponenten und Aufbau

Man braucht dafür:
* 1 Lego-Batteriebox
* 2 Lego M-Motor
* 1 Step-Down-Konverter
* 1 Motortreiber L298
* 1 Raspberry Pi Zero
* 2 Speedsensoren LM393 (angeklebt an Legoteil mit Achsloch)
* 2 Lego-Riemenscheiben als Lochscheibe
* ein kleines Breadboard (ebenfalls auf ein Legoteil geklebt)
* und diverse Lego-Technic-Teile

(( TODO: Foto von unten))

Die Motoren wurden nebeneinander angebracht, die Bewegung wird über Kegelzahnräder auf die Räder übertragen. Durch die dabei verwendete Untersetzung drehen sich die Räder langsamer als der Motor. Die Riemenscheiben wurden direkt mit den Motorachsen verbunden. Darüber wurden am Fahrgestell die Speedsensoren so befestigt, dass die Riemenscheiben sich innerhalb der Lichtschranke drehen.

(( TODO: Foto von Seite))

Die Batteriebox wurde oberhalb der Motoren angebracht und zwar so, dass sie sich gut auswechseln läßt.

Oben sind Raspi-Zero, Step-Down-Konverter und Motortreiber im Prototyp mit Gummis angebracht. Über den Step-Down-Konverter wird der Raspi über USB mit Strom aus der Batteriebox versorgt, der Motortreiber L298 wird über die Eingangspins des Step-Down-Konverters direkt an die Batteriebox angeschlossen.

Der Raspi steuert die über je drei Leitungen am Motortreiber L298 angeschlossenen Motoren. 

Die Stromversorgung der beiden Speedsensoren (3,3 V und Ground) läuft über das kleine Breadboard. Die D0-Pins der Speedsensoren werden mit den GPIO-Eingängen des Raspi verbunden.

Details der Verkabelung sind im folgenden Schaltplan zu finden.

(( TODO: Fritzing Schaltplan ))

## Programmierung

Bei der Programmierung sollen möglichst universelle Klassen zum Einsatz kommen.

Wir definieren dafür zunächst eine Klasse `SpeedSensMotor` die für Motoren mit Speedsensor verwendet werden kann.


### Klasse SpeedSensMotor für Motoren mit Speedsensor


Die Klasse SpeedSensMotor wird von der gpiozero-Klasse Moter abgeleitet und verfügt damit auch über deren Methoden.

Bei der Erzeugung von Objekten der Klasse SpeedSensMotor benötigt man neben den GPIO-Pin-Nummern über die der Motortreiber angeschlossen ist (in1_nr, in2_nr, ena_nr) noch folgende weitere Parameter:

* die Pinnummer des Speed-Sensors (speedsens_nr)
* die Anzahl der Löcher der Lochscheibe
* ggf. das Über- bez. Untersetzungsverhältnis beim Einsatz von Zahnrädern


```python
class SpeedSensMotor(Motor):

    def __init__(self, in1_nr, in2_nr, ena_nr, speedsens_nr, holes, gear_ratio=1):
        super(SpeedSensMotor, self).__init__(in1_nr, in2_nr, ena_nr)
        self.speedsensor_nr = speedsens_nr
        self.speedsensor = DigitalInputDevice(self.speedsensor_nr)
        self.speedsensor.when_activated = self.increment_counter
        self.speedsensor.when_deactivated = self.increment_counter
        self.reset_counter()
        self.holes = holes
        self.gear_ratio = gear_ratio
        # ab welcher Geschwindigkeit muss Bremsweg mit einberechnet werden
        self.brake_speed_threshold = .7

    def reset_counter(self):
        self.counter = 0

    def increment_counter(self):
        self.counter = self.counter + 1
        print(self.speedsensor_nr, self.counter)

    def turn_forward(self, revolutions, speed = 1):

        #print("forward")
        counts = revolutions * self.holes * 2 / self.gear_ratio
        # Reduce counts for braking
        if speed >= self.brake_speed_threshold and counts > 2:
            counts = counts-1
        self.reset_counter()
        self.forward(speed)
        while self.counter < counts:
            pass
        self.stop()
        self.reset_counter()

    def turn_backward(self, revolutions, speed = 1):

        #print("backward")
        counts = revolutions * self.holes * 2 / self.gear_ratio
        if speed >= self.brake_speed_threshold and counts > 2:
            counts = counts-1
        self.reset_counter()
        self.backward(speed)
        while self.counter < counts:
            pass
        self.stop()
        self.reset_counter()
```        

((TODO: Verlinkung zur gpiozero-Bibliothek))

Im Konstruktor wird dann der Konstruktor der Superklasse (also der gpiozero-Motor-Klasse) aufgerufen. Daneben wird ein Objekt der gpiozero-Klasse DigitalInputDevice für den Speedsensor erzeugt und die Methode festgelegt, die bei einem Wechsel von 0 nach 1, bzw. bei einem Wechsel von 1 nach 0 ausgeführt wird. In beiden Fällen soll der Zähler erhöht werden, also die Methoden `increment_counter` aufrufen werden.

Mit der Methode `reset_counter` kann der Zähler zurückgesetzt werden.

Neben den durch die Vererbung von der Klasse Motor bereits vorhandenen Methoden `forward` und `backward` zum dauerhaften Vorwärts- bzw. Rückwärtsdrehen, definieren wir zwei zusätzliche Methoden `turn_forward` und `turn_backward` um den Motor um eine bestimmte Anzahl von Umdrehungen (revolutions) vorwärts bzw. rückwärts zu drehen. Die Anzahl der Umdrehungen wird als Parameter mitgegeben, auch die Geschwindigkeit (zwischen 0 und 1) kann angegeben werden. 

Beim Aufruf der Methode `turn_forward` wird dann aus der Anzahl der Umdrehungen  mit der Anzahl der Löcher der Lochscheibe und dem Übersetzungsverhältnis die Anzahl der benötigten Wechsel von 0 nach 1 bzw. 1 nach 0 berechnet. Bei höhren Geschwindigkeiten wird für den Bremsweg noch eins abgezogen.

Nach Rücksetzen des Zählers wird der Motor über den Aufruf der Methode `forward` gestartet und solange gewartet, bis der Zähler die erforderliche Anzahl von Wechseln erreicht hat. Dann wird der Motor gestoppt.

Das Rückwärtsdrehen funktioniert genauso.

### Klasse SpeedSensRobot für Zwei-Motor-Roboter mit Speedsensoren


Diese Klasse leiten wir der Einfachheit halber nicht von der gpiozero-Klasse `Robot` ab, sondern definieren diese unabhängig davon.

Bei der Erzeugung von Objekten der Klasse SpeedSensRobot benötigt man folgende Parameter:

* ein Objekt der Klasse SpeedSensMotor als linken Motor
* ein Objekt der Klasse SpeedSensMotor als rechten Motor
* die Spurweite (track_width)
* den Radumfang (wheel_circumference)


((TODO Code))

Die Klasse hat folgende Methoden:
* `drive_forward` um eine bestimmte Strecke vorwärts zu fahren
* `drive _backward` um eine bestimmte Strecke rückwärts zu fahren
* `turn_left` um um einen bestimmten Winkel nach links zu drehen
* `turn_right` um um einen bestimmten Winkel nach rechts zu drehen

Bei allen Methoden wird zunächst berechnet, um wieviele Wechsel (0 nach 1 und 1 nach 0) sich jeder der Motoren drehen muss.

Beim Vorwärts- und Rückwärtsfahren werden hierzu zunächst aus der gewünschten Strecke und dem Radumfang die erforderlichen Umdrehungen und daraus dann die Anzahl der Wechsel berechnet. Auch hier wird bei höheren Geschwindigkeiten für den Bremsweg eins abgezogen.

Bei der Drehung dreht sich ein Motor vorwärts und der andere rückwärts. Hier gilt für die zu fahrende Strecke:

Strecke = Spurweite * 3,14 * winkel / 360

Aus der Strecke kann man dann mit Hilfe des Radumfangs wieder die erforderlichen Umdrehungen und daraus dann die Anzahl der Wechsel berechnen. Auch hier wird bei höheren Geschwindigkeiten für den Bremsweg eins abgezogen.

Nach der Berechnung der Anzahl der Wechsel für jeden Motor wird der Zähler jedes Motors zurückgesetzt und die Motoren dann gestartet. 

Dann wird in eine Schleife für jeden der beiden Motoren geprüft, ob die Anzahl der erforderlichen Wechsel schon erreicht ist und wenn dies der Fall ist, wird der Motor gestoppt.


### Nutzung der Klassen und Feinabstimmung

Für das oben beschriebene Fahrzeug kann man ein Objekt der Klasse SpeedSensRobot wie folgt erzeugen:

```python 
motor1 = SpeedSensMotor(in1_nr = 24,
                        in2_nr = 23, 
                        ena_nr = 26,
                        speedsens_nr = 4,
                        holes = 6,
                        gear_ratio .6)

motor2 = SpeedSensMotor(in1_nr = 16,
                        in2_nr = 20, 
                        ena_nr = 19,
                        speedsens_nr = 27,
                        holes = 6,
                        gear_ratio .6)

robot = SpeedSensRobot(left_motor = motor1,
                        right_motor = motor2,
                        track_width = 11.2,
                        wheel_cirumference = 15.4)
```

Beim Aufruf der Methoden zeigen sich beim Nachmessen eventuell Abweichungen, die verschiedene Ursachen (Bodenbeschaffenheit etc.) haben können. 

Für eine Feinabstimmung kann man dass Fahrzeug z.b. 100 cm vorwärts fahren lassen und dann nachzumessen. Durch Veränderung des Radumfangs kann die gefahrene Strecke angepasst werden.

```python 
robot.drive_forward(100,1)
```

Ähnlich kann man beim Drehen, das Fahrzeug um 360 Grad drehen lassen. Durch Veränderung der Spurweite kann dann die Drehung angepasst werden.

```python 
robot.turn_left(angle=360)
```
TODO: Programmcode auf Github


### Erkennung von Problemen

Wenn das Fahrzeug gegen eine Wand fährt, oder aus sonstigen Gründen nicht mehr weiterfahren kann, obwohl sich noch ein oder beide Motoren drehen, kann dies ebenfalls durch den Speedsensor erkannt werden.