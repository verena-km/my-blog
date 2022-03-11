---
layout: post
title:  "Raspi-Fahrzeug mit Speedsensor"
date:   2021-09-16
tags: Speedsensor Robot Raspberry Beispielfahrzeug
---

* TOC
{:toc}

Dieses Fahrzeug baut auf dem [Raspi-Fahrzeug mit zwei Antriebsmotoren]({% post_url 2021-09-21-raspi-car-gpiozero%}) auf.

Auch bei diesem Fahrzeug wird jedes der beiden Räder von einem Motor angetrieben. Zusätzlich ist an jedem der beiden Motoren ist ein Speedsensor LM393 angebracht. Hinten hat es eine einfache Stütze.

Durch den Speedsensor kann die Anzahl der Umdrehungen jedes Motors gemessen werden. Dadurch kann das Fahrzeug eine vorab bestimmte Strecke fahren bzw. sich um einen vorab festgelegten Winkel drehen. 

![Foto Fahrzeug mit zwei Antriebsmotoren und Speedsensoren](/images/foto_fahrzeug_speedsensor.jpg)

## Komponenten und Aufbau

Man braucht dafür:
* 1 Lego-Batteriebox
* 2 Lego M-Motoren
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

Der Raspi steuert die über je zwei Kabel am Motortreiber L298 angeschlossenen Motoren. Geschwindigkeitsänderungen können beim L298 entweder über ein PWM-Signal an den beiden IN-Eingängen oder über ein PWM-Signal am EN-Eingang erreicht werden. Da bei der Nutzzung von `gpiozero` sowieso das PWM-Signal an den IN-Eingängen verwendet wird, benötigen wir keine Verbindung vom Raspi zu den EN-Eingängen, sondern können diese durch Setzen der Jumper auf 'enabled' setzen.

Die Stromversorgung der beiden Speedsensoren (3,3 V und Ground) läuft über das kleine Breadboard vorne. Die D0-Pins der Speedsensoren werden mit den GPIO-Eingängen des Raspi verbunden.

Details der Verkabelung sind im folgenden Schaltplan zu finden.

![Schaltplan Fahrzeug mit Speedsensoren](/images/fritzing_fahrzeug_speedsensor.png)

## Programmierung

Bei der Programmierung sollen möglichst universelle Klassen zum Einsatz kommen.

Wir definieren dafür zunächst eine Klasse `SpeedSensMotor` die für Motoren mit Speedsensor verwendet werden kann.


### Klasse SpeedSensMotor für Motoren mit Speedsensor

Die Klasse SpeedSensMotor wird von der [gpiozero-Klasse Motor](https://gpiozero.readthedocs.io/en/stable/api_output.html#motor) abgeleitet und verfügt damit auch über deren Methoden.

Bei der Erzeugung von Objekten der Klasse `SpeedSensMotor` benötigt man neben den GPIO-Pin-Nummern über die der L298 angeschlossen ist (`in1_nr`, `in2_nr`) noch folgende weitere Parameter:

* die Pinnummer des Speed-Sensors (speedsens_nr)
* die Anzahl der Löcher der Lochscheibe (holes)
* ggf. das Über- bez. Untersetzungsverhältnis beim Einsatz von Zahnrädern (gear_ratio)
* ggf. die Geschwindigkeit, ab der der Bremsweg einberechnet werden soll (speed).

```python
from gpiozero import DigitalInputDevice
from gpiozero import Motor

class SpeedSensMotor(Motor):
    """
    Extends :class: `Motor` to use a motor with a simple speedsensor

    Attach the punched disk to the motor axle so it can rotate in the light barrier.
    Connect power 3.3V and ground of the speedsensor to the Pi and connect D0 to a free 
    GPIO pin.

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
        gear_ratio < 1: driven wheel faster
        gear_ratio > 1: driven wheel slower

    :break_speed_threshold:
        if speed is faster then break_speed_threshold
        stop earlier for breaking distance

    """
    def __init__(self, in1_nr, in2_nr, speedsens_nr, holes, gear_ratio=1, brake_speed_threshold=.7):
        super().__init__(in1_nr, in2_nr)
        self.speedsensor_nr = speedsens_nr
        self.speedsensor = DigitalInputDevice(self.speedsensor_nr)
        # Callback functions for speed sensor state changes 
        self.speedsensor.when_activated = self.increment_counter
        self.speedsensor.when_deactivated = self.increment_counter
        self.reset_counter()
        self.holes = holes
        self.gear_ratio = gear_ratio
        self.brake_speed_threshold = brake_speed_threshold

    def get_counts(self, revolutions, speed = None):
        """
            Returns the calculated counts for the punched disk from revolutions.
            If speed > brake_speed_threshold the calculated counts are reduced by one

            :param revolutions:
                the number of revolutions to convert
            :param speed:
                the motor speed
        """
        counts = revolutions * self.holes * 2 * self.gear_ratio
        if speed:
            if speed >= self.brake_speed_threshold and counts > 2:
                counts = counts-1
        return counts

    def reset_counter(self):
        """
            Reset the counter
        """
        self.counter = 0

    def increment_counter(self):
        """
            Increment the counter by 1
        """
        self.counter = self.counter + 1
        #print(self.speedsensor_nr, self.counter)

    def forward(self, speed=1, revolutions=None, counts=None):
        """
            Turn the motor forward

            :param float speed:
                The speed at which the motor should turn. Can be any value between
                0 (stopped) and the default 1 (maximum speed)
            :param float revolutions:
                The number of revolutions to turn - including gear ratio. This parameter can only be specified as a
                keyword parameter, and is mutually exclusive with *counts*
            :param int counts:
                The number of counts on the to turn.  This parameter can only be specified as a
                keyword parameter, and is mutually exclusive with *revolutions*
        """        
        if revolutions and counts:
            raise ValueError("Revolutions and counts can't be used at the same time.")

        if revolutions == None and counts == None:
            super().forward(speed)

        else:
            if revolutions:
                counts = self.get_counts(revolutions,speed)
            self.reset_counter()
            self.forward(speed)
            while self.counter < counts:
                pass
            self.stop()
            self.reset_counter()
     
    def backward(self, speed=1, revolutions=None, counts=None):
        """
            Turn the motor backward

            :param float speed:
                The speed at which the motor should turn. Can be any value between
                0 (stopped) and the default 1 (maximum speed)
            :param float revolutions:
                The number of revolutions to turn - including gear ratio. This parameter can only be specified as a
                keyword parameter, and is mutually exclusive with *counts*
            :param int counts:
                The number of counts on the to turn.  This parameter can only be specified as a
                keyword parameter, and is mutually exclusive with *revolutions*
        """        
        if revolutions and counts:
            raise ValueError("revolutions and counts can't be used at the same time")

        if revolutions == None and counts == None:
            super().backward(speed)

        else:
            if revolutions:
                counts = self.get_counts(revolutions,speed)
            self.reset_counter()
            self.backward(speed)
            while self.counter < counts:
                pass
            self.stop()
            self.reset_counter()

```        

Im Konstruktor wird dann der Konstruktor der Superklasse (also der gpiozero-Motor-Klasse) aufgerufen. Daneben wird ein Objekt der gpiozero-Klasse [DigitalInputDevice](https://gpiozero.readthedocs.io/en/stable/api_input.html#digitalinputdevice) für den Speedsensor erzeugt und die Methode festgelegt, die bei einem Wechsel von 0 nach 1, bzw. bei einem Wechsel von 1 nach 0 ausgeführt wird. In beiden Fällen soll der Zähler erhöht werden, also die Methoden `increment_counter` aufrufen werden. Mit der Methode `reset_counter` kann der Zähler zurückgesetzt werden.

Die von der Klasse `Motor` geerbten Methoden `forward` und `backward` zum Vorwärts- bzw. Rückwärtsdrehen werden überschrieben und erhalten zwei Schlüsselwortparameter `revolutions` und `counts`. Ruft man die Methoden mit dem Parameter `counts` auf, dreht der Motor um die entsprechende Anzahl Positionen der Lochscheibe. Ruft man die Methoden mit dem Parameter `revolutions` auf, so dreht sich der Motor um die entsprechende Anzahl der Umdrehungen. Wird keiner der Parameter angegeben, wird die entsprechende Methode der Klasse `Motor` aufgerufen - damit dreht sich der Motor dauerhaft.

Gibt man die Umdrehungen an, wird zunächst mit `get_counts` die Anzahl der zu zählenden Wechsel von 0 nach eins und umgekehrt berechnet. Dabei werden ggf. das Über- bez. Untersetzungsverhältnis bei der Verwendung von Zahnrädern sowie der Bremsweg berücksichtigt. Gibt man die direkt die Positionen an, ist keine Berechnung erforderlich. Nach Rücksetzen des Zählers wird der Motor über den Aufruf der Methode `forward` gestartet und solange gewartet, bis der Zähler die erforderliche Anzahl von Wechseln erreicht hat. Dann wird der Motor gestoppt.


### Klasse SpeedSensRobot für Zwei-Motoren-Roboter mit Speedsensoren

Diese Klasse leiten wir von der gpiozero-Klasse `Robot` ab. Grundlage ist allerdings die neueste Version von `Robot` auf Github, da deren Konstruktur zwei Motor-Objekte (und nicht mehr ein Zahlentupel) als Parameter verwendet. 

Bei der Erzeugung von Objekten der Klasse SpeedSensRobot benötigt man folgende Parameter:

* ein Objekt der Klasse SpeedSensMotor als linken Motor
* ein Objekt der Klasse SpeedSensMotor als rechten Motor
* die Spurweite (track_width)
* den Radumfang (wheel_circumference)
* ggf. Parameter zum Angleichen unterschiedlicher Motor-Geschwindigkeiten

```python
class SpeedSensRobot(NewRobot):
    """
    Dual-motor robot with speed-sensor at each motor

    :param left
        A :class: `SpeedSensMotor` for the left wheel of the robot

    :param right
        A :class: `SpeedSensMotor` for the right wheel of the robot

    :param float track_width
        The track width in cm

    :param float wheel_circumference
        The wheel_circumference in cm

    :speed_adjustment_slowdown
        slowdown factor to adjust diffrent motor speeds

    :adjustment_difference
        difference between left and right counter
        from which speed is adjusted
    """

    def __init__(self, left, right, track_width, wheel_circumference, speed_adjustment_slowdown =.9, adjustment_difference= 2, **kwargs ):
        super().__init__(left, right, **kwargs)
        self.track_width = track_width
        self.wheel_cirumference = wheel_circumference
        self.adjustment_slowdown = speed_adjustment_slowdown
        self.adjustment_difference = adjustment_difference

    def reset_counters(self):
        """
            Reset the counters of both motors
        """
        self.left_motor.reset_counter()
        self.right_motor.reset_counter()

    @property
    def counters(self):
        """
            Return the counters of both motors as tuple
        """
        return ((self.left_motor.counter,self.right_motor.counter))

    def forward(self, speed = 1, distance = None, **kwargs): 
        """
        Drive the robot forward by running both motors forward.
        :param float speed:
            Speed at which to drive the motors, as a value between 0 (stopped)
            and 1 (full speed). The default is 1.
        :param float curve_left:
            The amount to curve left while moving forwards, by driving the
            left motor at a slower speed. Maximum *curve_left* is 1, the
            default is 0 (no curve). This parameter can only be specified as a
            keyword parameter, and is mutually exclusive with *curve_right*.
        :param float curve_right:
            The amount to curve right while moving forwards, by driving the
            right motor at a slower speed. Maximum *curve_right* is 1, the
            default is 0 (no curve). This parameter can only be specified as a
            keyword parameter, and is mutually exclusive with *curve_left*.
        :param float distance:
            Distance to drive in cm. This parameter can only be specified as a
            keyword parameter, and is mutually exclusive with *curve_left* and 
            *curve_right*
        """
        if distance == None:
            # normal forward behavior from Robot class
            super().forward(speed, **kwargs)
        else:
            if "curve_left" in kwargs or "curve_right" in kwargs:
                raise ValueError("If distance is specicfied, curve is not possible.")
            else:
                revolutions = distance / self.wheel_cirumference
                counts_left = self.left_motor.get_counts(revolutions,speed)
                counts_right = self.right_motor.get_counts(revolutions,speed)
                   
                self.reset_counters()
                self.left_motor.forward(speed)
                self.right_motor.forward(speed)
                while self.left_motor.is_active or self.right_motor.is_active:
                    sleep(.01)
                    # adjust speed if one motor is faster
                    if self.left_motor.counter - self.right_motor.counter > self.adjustment_difference:
                        print("left too fast")
                        self.left_motor.forward(speed*self.adjustment_slowdown)
                        self.right_motor.forward(speed)                
                    if self.right_motor.counter - self.left_motor.counter > self.adjustment_difference:
                        print("right too fast")
                        self.right_motor.forward(speed*self.adjustment_slowdown)
                        self.left_motor.forward(speed)
                    # stop if counts are reached
                    if self.left_motor.counter >= counts_left:               
                        self.left_motor.stop()
                        print("left stopped")
                    if self.right_motor.counter >= counts_right:
                        self.right_motor.stop()
                        print("right stopped")

    def backward(self, speed=1, distance = None, **kwargs):        
        """
        Drive the robot backward by running both motors backward.
        :param float speed:
            Speed at which to drive the motors, as a value between 0 (stopped)
            and 1 (full speed). The default is 1.
        :param float curve_left:
            The amount to curve left while moving forwards, by driving the
            left motor at a slower speed. Maximum *curve_left* is 1, the
            default is 0 (no curve). This parameter can only be specified as a
            keyword parameter, and is mutually exclusive with *curve_right*.
        :param float curve_right:
            The amount to curve right while moving forwards, by driving the
            right motor at a slower speed. Maximum *curve_right* is 1, the
            default is 0 (no curve). This parameter can only be specified as a
            keyword parameter, and is mutually exclusive with *curve_left*.
        :param float distance:
            Distance to drive in cm. This parameter can only be specified as a
            keyword parameter, and is mutually exclusive with *curve_left* and 
            *curve_right*
        """
        if distance == None:
            # normal backward behavior from Robot class
            super().backward(speed, **kwargs)
        else:
            if "curve_left" in kwargs or "curve_right" in kwargs:
                raise ValueError("If distance is specicfied, curve is not possible.")
            else:
                #speed = kwargs.get("speed",1)
                revolutions = distance / self.wheel_cirumference
                counts_left = self.left_motor.get_counts(revolutions,speed)
                counts_right = self.right_motor.get_counts(revolutions,speed)
                   
                self.reset_counters()
                self.left_motor.backward(speed)
                self.right_motor.backward(speed)

                while self.left_motor.is_active or self.right_motor.is_active:
                    sleep(.01)
                    # adjust speed if one motor is faster
                    if self.left_motor.counter - self.right_motor.counter > self.adjustment_difference:
                        print("left too fast")
                        self.left_motor.backward(speed*self.adjustment_slowdown)
                        self.right_motor.backward(speed)                
                    if self.right_motor.counter - self.left_motor.counter > self.adjustment_difference:
                        print("right too fast")
                        self.right_motor.backward(speed*self.adjustment_slowdown)
                        self.left_motor.backward(speed)

                    if self.left_motor.counter >= counts_left:
                        self.left_motor.stop()
                        print("left stopped")
                    if self.right_motor.counter >= counts_right:
                        self.right_motor.stop()
                        print("right stopped")                

    def left(self, speed=1, angle=None):
        """
        Make the robot turn left by running the right motor forward and left
        motor backward.
        :param float speed:
            Speed at which to drive the motors, as a value between 0 (stopped)
            and 1 (full speed). The default is 1.

        :param float angle:
                The angle in degree.            
        """
        if angle == None:
            # normal backward behavior from Robot class
            super().left(speed)
        else:
            if angle <= 0 :
                raise ValueError("angle must be > 0")
            else:
                distance = self.track_width * 3.14 * angle/360
                revolutions =  distance / self.wheel_cirumference
                counts_left = self.left_motor.get_counts(revolutions,speed)
                counts_right = self.right_motor.get_counts(revolutions,speed)

                # TODO: Do we need adjustment?
                self.reset_counters()
                self.left_motor.backward(speed)
                self.right_motor.forward(speed)
                while self.left_motor.is_active or self.right_motor.is_active:
                    if self.left_motor.counter >= counts_left:
                        self.left_motor.stop()
                        print("left stopped")
                    if self.right_motor.counter >= counts_right:
                        self.right_motor.stop()
                        print("right stopped")


    def right(self, speed=1, angle=None):
        """
        Make the robot turn left by running the right motor forward and left
        motor backward.
        :param float speed:
            Speed at which to drive the motors, as a value between 0 (stopped)
            and 1 (full speed). The default is 1.

        :param float angle:
                The angle in degree.            
        """
        if angle == None:
            # normal backward behavior from Robot class
            super().right(speed)
        else:
            if angle <= 0 :
                raise ValueError("angle must be > 0")
            else:
                distance = self.track_width * 3.14 * angle/360
                revolutions =  distance / self.wheel_cirumference
                counts_left = self.left_motor.get_counts(revolutions,speed)
                counts_right = self.right_motor.get_counts(revolutions,speed)
              
                # TODO: Do we need adjustment?
                self.reset_counters()
                self.left_motor.forward(speed)
                self.right_motor.backward(speed)
                while self.left_motor.is_active or self.right_motor.is_active:
                    if self.left_motor.counter >= counts_left:
                        self.left_motor.stop()
                        print("left stopped")
                    if self.right_motor.counter >= counts_right:
                        self.right_motor.stop()
                        print("right stopped")
```


Die Klasse überschreibt die folgenden Methoden der Klasse `Robot`:
* `forward` erhält einen weiteren optionalen Parameter `distance` um eine bestimmte Strecke vorwärts zu fahren
* `backward` erhält einen weiteren optionalen Parameter `distance` um eine bestimmte Strecke rückwärts zu fahren
* `left` erhält einen weiteren optionalen Parameter `angle` um um einen bestimmten Winkel nach links zu drehen
* `right` erhält einen weiteren optionalen Parameter `angle` um um einen bestimmten Winkel nach links zu drehen

Falls die Methoden ohne diese Parameter aufgerufen werden, wird die jeweilige Methode der Klasse `Robot` ausgeführt.

Sind die Parameter angegeben, wird zunächst berechnet, um wieviele Wechsel (0 nach 1 und 1 nach 0) sich jeder der Motoren drehen muss.

Beim Vorwärts- und Rückwärtsfahren werden hierzu zunächst aus der gewünschten Strecke und dem Radumfang die erforderlichen Umdrehungen und daraus dann die Anzahl der Wechsel berechnet. 

Bei der Drehung dreht sich ein Motor vorwärts und der andere rückwärts. Hier gilt für die zu fahrende Strecke:
```
Strecke = Spurweite * 3,14 * winkel / 360
```
Aus der Strecke kann man dann mit Hilfe des Radumfangs wieder die erforderlichen Umdrehungen und daraus dann die Anzahl der Wechsel berechnen.

Nach der Berechnung der Anzahl der Wechsel für jeden Motor wird der Zähler jedes Motors zurückgesetzt und die Motoren dann gestartet. Danach wird in eine Schleife für jeden der beiden Motoren geprüft, ob die Anzahl der erforderlichen Wechsel schon erreicht ist und wenn dies der Fall ist, wird der Motor gestoppt. 

Falls einer der Motoren schon mehr Wechsel hat als der andere wird er leicht gebremst, damit die Räder in etwa gleichzeitig zum Stehen kommen. 

Der Konstruktor der Klasse Fahrzeug hat dafür zwei zusätzliche optionale Parameter, die das Verhalten steuern:
Der Parameter `adjustment_difference` legt fest, ab welchem Unterschied zwischen beiden Countern der eine Motor langsamer werden soll. Der Parameter `adjustment_slowdown` bestimmt dann mit welchem Faktor der gewählten Geschwindigkeit der schnellere Motor verlangsamt werden soll.

Als praktikabel hat sich ein Unterschied in den Countern von 3 und ein Slowdown auf 90% erwiesen



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


