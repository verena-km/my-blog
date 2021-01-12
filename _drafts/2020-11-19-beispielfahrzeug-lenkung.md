---
layout: post
title:  "Ein Fahrzeug mit Lenkung und Raspberry Pi Zero - Fernsteuerung mit BlueDot"
date:   2020-09-18 07:08:49 +0200
tags: RasberryPi Servomotor BlueDot Differenzial
---

Dieses von einem Raspi Zero gesteuerte Fahrzeug wird nur von einem Motor angetrieben und nutzt für die Lenkung einen Servomotor. Die Fernsteuerung läuft über Bluetooth mit einem Android-Smartphone und der App BlueDot.

![Foto Fahrzeug mit einem Antriebsmotor und Servomotor zur Lenkung](/images/foto_modell_vierradfahrzeug.jpg)

## Komponenten und Aufbau

Man braucht dafür:
* 1 Lego-Batteriebox
* 1 Lego M-Motor
* 1 Servomotor (Y3009)
* 1 Step-Down-Konverter
* 1 Motortreiber
* 1 Raspberry Pi Zero
* und diverse Lego-Technic-Teile

Verwendet wird eine alte Lego-Lenkung (Teile 2790, 2791 und 2792), deren Achse über zwei Zahnräder mit dem Servo-Motor verbunden ist, um den Bewegungsbereich zu vergrößern. 

![Foto Servomotor zur Lenkung](/images/foto_servolenkung.jpg)

Der M-Motor treibt zunächst ein kleines Zahnrad, dieses ein größeres Zahnrad und dieses wiederum ein Differenzial an, das mit den beiden Hinterrädern verbunden ist. Durch das Differenzial werden die ungleichen Fahrwege links und rechts bei Kurven ausgeglichen. 

![Foto Differenzial](/images/foto_differenzial.jpg)

Über den Step-Down-Konverter wird der Raspi über USB mit Strom aus der Batteriebox versorgt, der Motortreiber L298 wird über die Eingangspins des Step-Down-Konverters direkt an die Batteriebox angeschlossen.

Der Raspi versorgt den Servomotor mit Strom (5V) und steuert diesen. Über drei weitere Leitungen steuert der Raspi den M-Motor am L298. Detail der Verkabelung sind im folgenden Schaltplan zu sehen:

![Schaltplan Fahrzeug mit Servo-Lenkung](/images/fritzing_raspi_servofahrzeug.png)

## Programmierung

Bei Programmierung des Systems kommen zum Einsatz
* die Klassen `Motor` und `AngularServo` aus der Bibliothek [gpiozero](https://gpiozero.readthedocs.io/en/stable/)
* die Bibliothek bzw. App [BlueDot](https://bluedot.readthedocs.io/en/latest/)

Mit BlueDot kann man einen Raspberry Pi von einem Android-Smartphone fernsteuern. Dazu benötigt man ein Smartphone, auf dem die BlueDot-App installiert ist. Dieses muss mit dem Raspi gepaired werden. Auf dem Raspi läuft ein Python-Programm unter Verwendung der BlueDot-Bibliothek. Aus der Bluedot-App kann man dann eine Verbindung zum Raspi aufbauen und bekommt die auf dem Raspi programmierten Steuermöglichkeiten angezeigt. Auf der BlueDot-Dokumentationsseite ist alles ausführlich und einfach beschrieben.

Für den Servomotor Y3009 nutzen wir die von `AngularServo` abgeleitete Klasse `Y3009` (siehe [Post zum Servomotor]({% post_url 2020-11-19-servo-motor %}) ) und definieren eine neue Klasse `ServoCar`, zu dessen Objekten ein normaler Motor und ein Servomotor gehören. Wir definieren weiter eine Methode drive, die als Parameter die Geschwindigkeit (zwischen -1 und 1, also auch rückwärts) und den Winkel (ebenfalls zwischen -1 und +1 erhält).

```python
from motors import Y3009
from gpiozero import Motor
from time import sleep

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
```

Das Modul zur Steuerung mit BlueDot ist dann nicht mehr sehr aufwändig:

```python
from cars import ServoCar
from motors import Y3009
from gpiozero import Motor
from bluedot import BlueDot
from signal import pause

def move(pos):
    if pos.y > 0:
        speed = pos.distance
    else:
        speed = -pos.distance
    my_car.drive(speed=speed, direction=pos.x)

def stop():
    my_car.stop()

def shutdown():
    print("shudown")

# Erzeugen des Motorobjekts unter Angabe der genutzten Pins
my_motor = Motor(forward=17,backward=27,enable=22)
# Erzeugen des Servoobjekts unter Angabe des genutzten Pins
my_servo = Y3009(pin=18)
# Erzeugen des Fahrzeugobjekts unter Angabe der beiden Motoren
my_car = ServoCar(my_servo, my_motor)

# BlueDot-Objekt erzeugen
bd = BlueDot(cols=1,rows=2)
bd[0,0].color = "blue"
bd[0,1].color = "red"
# Auszuführende Methoden festlegen
# beim Drücken
bd[0,0].when_pressed = move
# beim Bewegen
bd[0,0].when_moved = move
# beim Loslassen
bd[0,0].when_released = stop

bd[1,0].when_pressed = shutdown

pause()
```


## Python-Programm beim Systemstart starten

Damit die Fernsteuerung nach dem Hochfahren des Raspis direkt verfügbar ist, muss das Python-Programm beim Systemstart automatisch gestartet werden. Wie man hier gut nachlesen kann, gibt es mehrere Möglichkeiten zum automatischen Starten:

[https://www.dexterindustries.com/howto/run-a-program-on-your-raspberry-pi-at-startup/](https://www.dexterindustries.com/howto/run-a-program-on-your-raspberry-pi-at-startup/)

Ich wähle die Methode, den Programmstart in die Datei `/etc/rc.local` einzutragen. 

Folgender Eintrag wird der Datei in der vorletzten Zeile hinzugefügt. Die Zeile `exit 0` bleibt dahinter stehen:

```
...
sudo python3 /home/pi/python_robbi/bluedot_servocar.py &
exit 0
```








