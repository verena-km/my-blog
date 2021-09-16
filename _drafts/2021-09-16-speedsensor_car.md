---
layout: post
title:  "Speedsensor zur Motor- und Fahrzeugsteuerung"
date:   2021-09-16
tags: Speedsensor Robot Raspberry Beispielfahrzeug
---

Bei diesem Fahrzeug wird jedes der beiden Räder von einem Motor angetrieben. An jedem der beiden Motoren ist ein Speedsensor LM393 angebracht. Hinten hat es eine einfache Stütze.

Durch den Speedsensor kann die Anzahl der Umdrehungen jedes Motors gemessen werden. Dadurch kann das Fahrzeug eine vorab bestimmte Strecke fahren bzw. sich um einen vorab festgelegten Winkel drehen. 

![Foto Fahrzeug mit zwei Antriebsmotoren und Speedsensoren](/images/foto_fahrzeug_speedsensor.jpg)

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

Die Motoren wurden nebeneinander angebracht, die Bewegung wird über Kegelzahnräder auf die Räder übertragen. Durch die dabei verwendete Untersetzung drehen sich die Räder langsamer als der Motor. Die Riemenscheiben wurden direkt mit den Motorachsen verbunden. Darüber wurden am Fahrgestell die Speedsensoren so befestigt, dass die Riemenscheiben sich innerhalb der Lichtschranke drehen.

![Foto Fahrzeug Fahrzeug von unten](/images/foto_fahrzeug_speedsensor_unten.jpg)

Die Batteriebox wurde oberhalb der Motoren angebracht und zwar so, dass sie sich gut auswechseln läßt.

Oben sind Raspi-Zero, Step-Down-Konverter und Motortreiber im Prototyp mit Gummis angebracht. Über den Step-Down-Konverter wird der Raspi über USB mit Strom aus der Batteriebox versorgt, der Motortreiber L298 wird über die Eingangspins des Step-Down-Konverters direkt an die Batteriebox angeschlossen.

![Foto Fahrzeug Fahrzeug von oben](/images/foto_fahrzeug_speedsensor_oben.jpg)

Der Raspi steuert die über je zwei Kabel am Motortreiber L298 angeschlossenen Motoren. Geschwindigkeitsänderungen können beim L298 entweder über ein PWM-Signal an den beiden IN-Eingängen eines Motors oder über ein PWM-Signal am EN-Eingang erreicht werden. Da bei der Nutzzung von `gpiozero` sowieso das PWM-Signal an den IN-Eingängen verwendet wird, benötigen wir keine Verbindung vom Raspi zu den EN-Eingängen, sondern können diese durch Setzen der Jumper auf 'enabled' setzen.

Die Stromversorgung der beiden Speedsensoren (3,3 V und Ground) läuft über das kleine Breadboard vorne. Die D0-Pins der Speedsensoren werden mit den GPIO-Eingängen des Raspi verbunden.

Details der Verkabelung sind im folgenden Schaltplan zu finden.

![Schaltplan Fahrzeug mit Speedsensoren](/images/fritzing_fahrzeug_speedsensor.png)

## Programmierung

Bei der Programmierung sollen möglichst universelle Klassen zum Einsatz kommen.

Wir definieren dafür zunächst eine Klasse `SpeedSensMotor` die für Motoren mit Speedsensor verwendet werden kann.


### Klasse SpeedSensMotor für Motoren mit Speedsensor

Die Klasse SpeedSensMotor wird von der [gpiozero-Klasse Moter](https://gpiozero.readthedocs.io/en/stable/api_output.html#motor) abgeleitet und verfügt damit auch über deren Methoden.

Bei der Erzeugung von Objekten der Klasse SpeedSensMotor benötigt man neben den GPIO-Pin-Nummern über die der L298 angeschlossen ist (in1_nr, in2_nr) noch folgende weitere Parameter:

* die Pinnummer des Speed-Sensors (speedsens_nr)
* die Anzahl der Löcher der Lochscheibe
* ggf. das Über- bez. Untersetzungsverhältnis beim Einsatz von Zahnrädern

```python
from gpiozero import DigitalInputDevice
from gpiozero import Motor

class SpeedSensMotor(Motor):
    """
    Extends :class: `Motor` to use a motor with a simple speedsensor

    Attach the punched disk to the motor axle so it can rotate in the light
    barrier. Connect power 3.3V and ground of the speedsensor to the Pi and 
    connect D0 to a free GPIO pin.

    :param in1_nr:
        The first GPIO pin the motor driver is connected to.

    :param in2_nr:
        The second GPIO pin the motor driver is connected to.

    :param speedsens_nr:
        The GPIO pin the speed sensor D0 is connected to

    :param holes:
        The count of holes in the punched disk.

    :param gear_ratio:
        If gears are used this is the gear ratio
        gear_ratio > 1 - driven wheel faster
        gear_ratio < 1 - driven wheel slower

    """
    def __init__(self, in1_nr, in2_nr, speedsens_nr, holes, gear_ratio=1):
        super(SpeedSensMotor, self).__init__(in1_nr, in2_nr)
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

    def turn_forward(self, revolutions, speed = 1):
        """
            Turn the motor the given number of revolutions forward"
            :param float revolutions:
                The number of revolutions to turn
            :param float speed:
                The speed at witch the motor should turn. Can be
                any value between 0 (stopped) and the default 1 
                (maximum speed)
        """
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
        """
            Turn the motor the given number of revolutions forward"
            :param float revolutions:
                The number of revolutions to turn
            :param float speed:
                The speed at witch the motor should turn. Can be any
                value between 0 (stopped) and the default 1 (maximum 
                speed)
        """
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

Im Konstruktor wird dann der Konstruktor der Superklasse (also der gpiozero-Motor-Klasse) aufgerufen. Daneben wird ein Objekt der gpiozero-Klasse [DigitalInputDevice](https://gpiozero.readthedocs.io/en/stable/api_input.html#digitalinputdevice) für den Speedsensor erzeugt und die Methode festgelegt, die bei einem Wechsel von 0 nach 1, bzw. bei einem Wechsel von 1 nach 0 ausgeführt wird. In beiden Fällen soll der Zähler erhöht werden, also die Methoden `increment_counter` aufrufen werden.

Mit der Methode `reset_counter` kann der Zähler zurückgesetzt werden.

Neben den durch die Vererbung von der Klasse Motor bereits vorhandenen Methoden `forward` und `backward` zum dauerhaften Vorwärts- bzw. Rückwärtsdrehen, definieren wir zwei zusätzliche Methoden `turn_forward` und `turn_backward` um den Motor um eine bestimmte Anzahl von Umdrehungen (revolutions) vorwärts bzw. rückwärts zu drehen. Die Anzahl der Umdrehungen wird als Parameter mitgegeben, auch die Geschwindigkeit (zwischen 0 und 1) kann angegeben werden. 

Beim Aufruf der Methode `turn_forward` wird dann aus der Anzahl der Umdrehungen  mit der Anzahl der Löcher der Lochscheibe und dem Übersetzungsverhältnis die Anzahl der benötigten Wechsel von 0 nach 1 bzw. 1 nach 0 berechnet. Bei höhren Geschwindigkeiten wird für den Bremsweg noch eins abgezogen.

Nach Rücksetzen des Zählers wird der Motor über den Aufruf der Methode `forward` gestartet und solange gewartet, bis der Zähler die erforderliche Anzahl von Wechseln erreicht hat. Dann wird der Motor gestoppt.

Das Rückwärtsdrehen funktioniert genauso.

### Klasse SpeedSensRobot für Zwei-Motoren-Roboter mit Speedsensoren

Diese Klasse leiten wir der Einfachheit halber nicht von der gpiozero-Klasse `Robot` ab, sondern definieren diese unabhängig davon.

Bei der Erzeugung von Objekten der Klasse SpeedSensRobot benötigt man folgende Parameter:

* ein Objekt der Klasse SpeedSensMotor als linken Motor
* ein Objekt der Klasse SpeedSensMotor als rechten Motor
* die Spurweite (track_width)
* den Radumfang (wheel_circumference)

```python
class SpeedSensRobot():
    """
    Dual-motor robot with speed-sensor at each motor

    :param left_motor
        A :class: `SpeedSensMotor` for the left wheel of the robot

    :param right_motor
        A :class: `SpeedSensMotor` for the right wheel of the robot

    :param float track_width
        The track width in cm

    :param float wheel_circumference
        The wheel_circumference in cm
    """

    def __init__(self, left_motor, right_motor, track_width, wheel_circumference):
        self.left_motor = left_motor
        self.right_motor = right_motor
        self.track_width = track_width
        self.wheel_cirumference = wheel_circumference

    def drive_forward(self, distance, speed = 1):
        """
        Drive the robot forward for a specific distance.

            :param float distance:
                The distance in cm.

            :param speed
                The speed at witch the motors should turn. Can be any value 
                between 0 (stopped) and the default 1 (maximum speed).
        """
        revolutions = distance / self.wheel_cirumference
        counts_left = revolutions * self.left_motor.holes * 2 / self.left_motor.gear_ratio
        counts_right = revolutions * self.right_motor.holes * 2 / self.right_motor.gear_ratio

        # Reduce counts for braking
        if speed >= self.left_motor.brake_speed_threshold and counts_left > 2:
            counts_left = counts_left-1
        if speed >= self.right_motor.brake_speed_threshold and counts_right > 2:
            counts_right = counts_right-1                    

        self.left_motor.reset_counter()
        self.right_motor.reset_counter()
        self.left_motor.forward(speed)
        self.right_motor.forward(speed)
        while self.left_motor.is_active or self.right_motor.is_active:
            if self.left_motor.counter >= counts_left:               
                self.left_motor.stop()
            if self.right_motor.counter >= counts_right:
                self.right_motor.stop()

    def drive_backward(self, distance, speed = 1):
        """
        Drive the robot backward for a specific distance.

            :param float distance:
                The distance in cm.

            :param speed
                The speed at witch the motors should turn. Can be any value
                between 0 (stopped) and the default 1 (maximum speed).
        """      
        revolutions = distance / self.wheel_cirumference
        counts_left = revolutions * self.left_motor.holes * 2 / self.left_motor.gear_ratio
        counts_right = revolutions * self.right_motor.holes * 2 / self.left_motor.gear_ratio

        # Reduce counts for braking
        if speed >= self.left_motor.brake_speed_threshold and counts_left > 2:
            counts_left = counts_left-1
        if speed >= self.right_motor.brake_speed_threshold and counts_right > 2:
            counts_right = counts_right-1           

        self.left_motor.reset_counter()
        self.right_motor.reset_counter()
        self.left_motor.backward(speed)
        self.right_motor.backward(speed)
        while self.left_motor.is_active or self.right_motor.is_active:
            if self.left_motor.counter >= counts_left:
                self.left_motor.stop()
            if self.right_motor.counter >= counts_right:
                self.right_motor.stop()

    def turn_left(self, angle=90, speed = 1):
        """
        Turn the robot left for the given angle.

            :param float angle:
                The angle in degree.

            :param speed
                The speed at witch the motors should turn. Can be any value
                between 0 (stopped) and the default 1 (maximum speed).
        """
        distance = self.track_width * 3.14 * angle/360
        revolutions =  distance / self.wheel_cirumference
        counts_left = revolutions * self.left_motor.holes * 2 / self.left_motor.gear_ratio
        counts_right = revolutions * self.right_motor.holes * 2 / self.left_motor.gear_ratio

        # Reduce counts for braking
        if speed >= self.left_motor.brake_speed_threshold and counts_left > 2:
            counts_left = counts_left-1
        if speed >= self.right_motor.brake_speed_threshold and counts_right > 2:
            counts_right = counts_right-1

        
        self.left_motor.reset_counter()
        self.right_motor.reset_counter()
        self.left_motor.backward(speed)
        self.right_motor.forward(speed)
        while self.left_motor.is_active or self.right_motor.is_active:
            if self.left_motor.counter >= counts_left:
                self.left_motor.stop()
            if self.right_motor.counter >= counts_right:
                self.right_motor.stop()

    def turn_right(self, angle=90, speed = 1):
        """
        Turn the robot right for the given angle.

            :param float angle:
                The angle in degree.

            :param speed
                The speed at witch the motors should turn. Can be any value
                between 0 (stopped) and the default 1 (maximum speed).
        """
        distance = self.track_width * 3.14 * angle/360        
        revolutions =  distance / self.wheel_cirumference
        counts_left = revolutions * self.left_motor.holes * 2 / self.left_motor.gear_ratio
        counts_right = revolutions * self.right_motor.holes * 2 / self.left_motor.gear_ratio

        # Reduce counts for braking
        if speed >= self.left_motor.brake_speed_threshold and counts_left > 2:
            counts_left = counts_left-1
        if speed >= self.right_motor.brake_speed_threshold and counts_right > 2:
            counts_right = counts_right-1

        self.left_motor.reset_counter()
        self.right_motor.reset_counter()
        self.left_motor.forward(speed)
        self.right_motor.backward(speed)
        while self.left_motor.is_active or self.right_motor.is_active:
            if self.left_motor.counter >= counts_left:
                self.left_motor.stop()
            if self.right_motor.counter >= counts_right:
                self.right_motor.stop()
```


Die Klasse hat folgende Methoden:
* `drive_forward` um eine bestimmte Strecke vorwärts zu fahren
* `drive _backward` um eine bestimmte Strecke rückwärts zu fahren
* `turn_left` um um einen bestimmten Winkel nach links zu drehen
* `turn_right` um um einen bestimmten Winkel nach rechts zu drehen

Bei allen Methoden wird zunächst berechnet, um wieviele Wechsel (0 nach 1 und 1 nach 0) sich jeder der Motoren drehen muss.

Beim Vorwärts- und Rückwärtsfahren werden hierzu zunächst aus der gewünschten Strecke und dem Radumfang die erforderlichen Umdrehungen und daraus dann die Anzahl der Wechsel berechnet. Auch hier wird bei höheren Geschwindigkeiten für den Bremsweg eins abgezogen.

Bei der Drehung dreht sich ein Motor vorwärts und der andere rückwärts. Hier gilt für die zu fahrende Strecke:
```
Strecke = Spurweite * 3,14 * winkel / 360
```
Aus der Strecke kann man dann mit Hilfe des Radumfangs wieder die erforderlichen Umdrehungen und daraus dann die Anzahl der Wechsel berechnen. Auch hier wird bei höheren Geschwindigkeiten für den Bremsweg eins abgezogen.

Nach der Berechnung der Anzahl der Wechsel für jeden Motor wird der Zähler jedes Motors zurückgesetzt und die Motoren dann gestartet. 

Dann wird in eine Schleife für jeden der beiden Motoren geprüft, ob die Anzahl der erforderlichen Wechsel schon erreicht ist und wenn dies der Fall ist, wird der Motor gestoppt.


### Nutzung der Klassen und Feinabstimmung

Für das oben beschriebene Fahrzeug kann man ein Objekt der Klasse SpeedSensRobot wie folgt erzeugen:

```python 
motor_links = SpeedSensMotor(in1_nr = 24,
                        in2_nr = 23, 
                        speedsens_nr = 19,
                        holes = 6,
                        gear_ratio .6)

motor_rechts = SpeedSensMotor(in1_nr = 12,
                        in2_nr = 25, 
                        speedsens_nr = 26,
                        holes = 6,
                        gear_ratio .6)

robot = SpeedSensRobot(left_motor = motor_links,
                        right_motor = motor_rechts,
                        track_width = 11.2,
                        wheel_cirumference = 15.4)
```

Beim Aufruf der Methoden zeigen sich beim Nachmessen eventuell Abweichungen, die verschiedene Ursachen (Bodenbeschaffenheit etc.) haben können. 

Für eine Feinabstimmung kann man dass Fahrzeug z.B. 100 cm vorwärts fahren lassen und dann nachzumessen. Durch Veränderung des Radumfangs kann die gefahrene Strecke angepasst werden.

```python 
robot.drive_forward(100,1)
```

Ähnlich kann man beim Drehen, das Fahrzeug um 360 Grad drehen lassen. Durch Veränderung der Spurweite kann dann die Drehung angepasst werden.

```python 
robot.turn_left(angle=360)
```

Sobald man das gemacht hat, kann man den Roboter beispielsweise im Quadrat fahren lassen.

```python 
for i in range(4):
    robot.drive_forward(30)
    robot.turn_right(90)
```

Der Grundaufbau des Fahrzeugs und die erweiterten Klassen können nun als Basis für weitere Projekte mit autonomen oder ferngesteuerten Robotern verwendet werden.