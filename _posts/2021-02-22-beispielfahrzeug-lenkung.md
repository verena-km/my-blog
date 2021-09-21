---
layout: post
title:  "Ein Fahrzeug mit Lenkung und Raspberry Pi Zero - Fernsteuerung mit BlueDot"
date:   2021-02-22 
tags: Raspberry Servomotor BlueDot Differenzial Systemstart Fernsteuerung gpiozero Motor L298
---

Dieses von einem Raspi Zero gesteuerte Fahrzeug wird von einem Motor angetrieben und nutzt für die Lenkung einen Servomotor. Es kann von einem Android-Smartphone mit der App BlueDot über Bluetooth ferngesteuert werden.

![Foto Fahrzeug mit einem Antriebsmotor und Servomotor zur Lenkung](/images/foto_modell_vierradfahrzeug.jpg)

## Komponenten und Aufbau

Man braucht dafür:
* 1 Lego-Batteriebox
* 1 Lego M-Motor
* 1 270°-Servomotor (grauer Geekservo)
* 1 Step-Down-Konverter
* 1 Motortreiber L298
* 1 Raspberry Pi Zero
* und diverse Lego-Technic-Teile

Verwendet wird eine alte Lego-Lenkung (Teile 2790, 2791 und 2792), deren Achse mit einem Geekservo verbunden ist. 

![Foto Servomotor zur Lenkung](/images/foto_servolenkung.jpg)

Der Lego M-Motor treibt über ein Zahnrad ein Differenzial an, das mit den beiden Hinterrädern verbunden ist. Durch das Differenzial werden die ungleichen Fahrwege links und rechts bei Kurven ausgeglichen. 

![Foto Differenzial](/images/foto_differenzial.jpg)

Über den Step-Down-Konverter wird der Raspi über USB mit Strom aus der Batteriebox versorgt, der Motortreiber L298 wird über die Eingangspins des Step-Down-Konverters direkt an die Batteriebox angeschlossen.

Der Raspi versorgt den Servomotor mit Strom (5V) und steuert diesen. Über drei weitere Leitungen steuert der Raspi den M-Motor am L298. Detail der Verkabelung sind im folgenden Schaltplan zu sehen:

![Schaltplan Fahrzeug mit Servo-Lenkung](/images/fritzing_raspi_servofahrzeug.png)

## Programmierung

Bei Programmierung des Systems kommen zum Einsatz
* die Klassen `Motor` und `AngularServo` aus der Bibliothek [gpiozero](https://gpiozero.readthedocs.io/en/stable/)
* die Bibliothek bzw. App [BlueDot](https://bluedot.readthedocs.io/en/latest/)

Mit BlueDot kann man einen Raspberry Pi von einem Android-Smartphone fernsteuern. Dazu benötigt man ein Smartphone, auf dem die BlueDot-App installiert ist. Dieses muss mit dem Raspi gepaired werden. Auf dem Raspi läuft ein Python-Programm unter Verwendung der BlueDot-Bibliothek. Aus der BlueDot-App kann man dann eine Verbindung zum Raspi aufbauen und bekommt die auf dem Raspi programmierten Steuermöglichkeiten angezeigt. In der [BlueDot-Dokumentation](https://bluedot.readthedocs.io) ist alles ausführlich und einfach beschrieben.

Für den grauen Geekservo nutzen wir eine von `AngularServo` abgeleitete Klasse `GeekServoGray` (siehe hierzu auch [Post zum Servomotor]({% post_url 2020-11-19-servo-motor %}) ) und definieren eine neue Klasse `ServoCar`, zu dessen Objekten ein normaler Motor und ein Servomotor gehören. Wir definieren weiter eine Methode `drive`, die als Parameter die Geschwindigkeit (zwischen -1 und 1, also auch rückwärts) und den Winkel (ebenfalls zwischen -1 und +1 erhält).

Hier nun das Python-Programm `bluedot_servocar.py`:

```python
from bluedot import BlueDot
from signal import pause

from gpiozero import Motor, AngularServo
from gpiozero.pins.pigpio import PiGPIOFactory

from time import sleep

factory = PiGPIOFactory()

class GeekServoGray(AngularServo):
    def __init__(self, pin):
         super().__init__(pin,
                            min_angle=-145,
                            max_angle=145,
                            min_pulse_width=0.5/1000,
                            max_pulse_width=2.5/1000,
                            pin_factory = factory )

class ServoCar:
    def __init__(self, servo_motor, dc_motor):
        self.servo_motor = servo_motor
        self.dc_motor = dc_motor

    def drive(self, speed=1, direction=0):
        # Umrechnung Einschlag des Lenkrads (keins=0, links=-1, rechts=1)
        self.servo_motor.angle = direction*self.servo_motor.max_angle

        # gpiozero kennt keie negative Geschwindigkeit
        if speed > 0:
            self.dc_motor.forward(speed)
        else:
            self.dc_motor.backward(abs(speed))

    def stop(self):
        self.drive(speed=0)

# Fahrbewegung anhand der Bluedot-Position durchführen
# die y-Position ergibt die Geschwindigkeit, die x-Position die Richtung
def move(pos):
    my_car.drive(speed=pos.y, direction=pos.x)

# Fahrzeug stoppen
def stop():
    my_car.stop()

# Raspi herunterfahren
def shutdown():
    command = "/usr/bin/sudo /sbin/shutdown -r now"
    import subprocess
    process = subprocess.Popen(command.split(), stdout=subprocess.PIPE)
    output = process.communicate()[0]

# Erzeugen des Motorobjekts unter Angabe der genutzten Pins
my_motor = Motor(forward=17,backward=27,enable=22)
# Erzeugen des Servoobjekts unter Angabe des genutzten Pins
my_servo = GeekServoGray(pin=18)
# Erzeugen des Fahrzeugobjekts unter Angabe der beiden Motoren
my_car = ServoCar(my_servo, my_motor)

# BlueDot-Objekt erzeugen
bd = BlueDot(cols=1,rows=2)
bd[0,0].color = "blue"
bd[0,0].square = True
bd[0,1].color = "red"
# Auszuführende Methoden festlegen
# beim Drücken
bd[0,0].when_pressed = move
# beim Bewegen
bd[0,0].when_moved = move
# beim Loslassen
bd[0,0].when_released = stop
# beim Drücken des roten Buttons
bd[0,1].when_pressed = shutdown

pause()
```

Für die Steuerung wird ein BlueDot-Objekt mit einer Spalte und zwei Reihen erzeugt und darin ein blaues Quadrat zur Steuerung und ein roter Kreis zum Herunterfahren des Raspis. Beim Berühren und bei Bewegungen im blauen Quadrat in der BlueDot-App auf dem Smartphone werden die Koordinaten der Position an den Raspi übermittelt. Das sind sowohl auf der X-Achse, als auch auf der Y-Achse Werte zwischen -1 und 1. Diese können dann ohne Umrechnen von der drive-Methode zur Fahrzeugsteuerung genutzt werden: Der x-Wert wird zur Richtung, der y-Wert zur Geschwindigkeit bzw. Fahrtrichtung. Sobald man das blaue Quadrat nicht mehr berührt, stoppt das Fahrzeug.

Hier ein Screenshot der App:

![Screenshot Smartphone BlueDot](/images/screenshot_bluedot.png)


## Python-Programm beim Systemstart starten

Damit die Fernsteuerung nach dem Hochfahren des Raspis direkt verfügbar ist, muss das Python-Programm beim Systemstart automatisch gestartet werden. Wie man hier gut nachlesen kann, gibt es mehrere Möglichkeiten zum automatischen Starten: [https://www.dexterindustries.com/howto/run-a-program-on-your-raspberry-pi-at-startup/](https://www.dexterindustries.com/howto/run-a-program-on-your-raspberry-pi-at-startup/)

Ich wähle die Methode, den Programmstart in die Datei `/etc/rc.local` einzutragen. 

Folgender Eintrag wird der Datei in der vorletzten Zeile hinzugefügt. Die Zeile `exit 0` bleibt dahinter stehen:

```
...
sudo python3 /home/pi/python_robbi/bluedot_servocar.py &
exit 0
```







