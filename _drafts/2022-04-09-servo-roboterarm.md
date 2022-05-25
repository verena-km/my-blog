---
layout: post
title:  "Roboterarm mit Servos"
date:   2022-04-30
tags: Microcontroller
---

In diesem Projekt bauen wir aus vier Servomotoren und ein paar Legoteilen einen einfachen Roboterarm. Zur Steuerung kommt ein Raspberry Pi Pico zum Einsatz.

## Komponenten und Aufbau

Vom KIT gibt es eine Vorlesung zu Robotik auf Youtube: 

https://www.youtube.com/playlist?list=PLfk0Dfh13pBMvdwxbVFeXmKWLyDcL2vSQ 

http://hades.mech.northwestern.edu/images/7/7f/MR.pdf

https://homepages.thm.de/~hg6458/Robotik/Robotik.pdf

https://www.robotshop.com/community/tutorials/show/basics-how-do-i-choose-a-robot-arm

https://appliedgo.net/roboticarm/ 


Ein Roboter(arm) besteht aus mehren Gliedern und Gelenken. Unserer hat den 

Freiheitsgrade


Wir benötigten:
* 1 Rapsberry Pi Pico
* 4 graue Geek-Servo
* 1 Breadboard
* einige Jumperkabel

((Foto))

Schaltbild

((Fritzing))


## Programmierung


### Klasse für Servo mit langsamen Bewegungen

Im [Beitrag über Servomotoren]({% post_url 2020-11-19-servo-motor %}) habe ich bereits eine Klasse für Servomotoren unter Micropython erstellt. 

Für den Pico müssen wir die Klasse etwas modifizieren, da die Micropython-Klasse PWM auf dem Pico die Methode duty() nicht kennt. Stattdessen verwenden wir die Methode duty_ns(), bei der man die Pulsweite in Nanosekunden als Argument angibt.

Eine weitere Änderung ergibt sich dadurch, dass wir für den Roboterarm langsame Bewegungen brauchen. Das kann man dadurch erreichen, dass man den Motor nur in Minischritten vorwärtsbewegt und dazwischen immer einen Moment pausiert.

Hierfür bekommt die Klasse die Methode `set_angle_slow`, die neben dem gewünschten Zielwinkel die Parameter `step_size` und `delay` hat. Mit `step_size` legt man fest um wieviel Grad sich der Motor in einem Schritt dreht und mit `delay` wie lange er bis zum nächsten Schritt pausiert. 

In der Methode wird zunächst aus der aktuelle Pulsweite der derzeitigen Winkel ermittelt und dann der Winkel der erforderlichen Drehung berechnet. Dann wird der Servo so lange um jeweils einen Schritt mit Pause in die entsprechende Richtung weiterbewegt, bis der Zielwinkel erreicht ist.


```python 
class AngularServo:
    def __init__(self, pin,
                          initial_angle = 0,
                          min_angle = -90,
                          max_angle = 90,
                          min_pulse_width = 1.0/1000,
                          max_pulse_width = 2.0/1000):
        self.servo = PWM(Pin(pin,Pin.OUT))
        self.servo.freq(50)
        self.angle = initial_angle
        self.min_angle = min_angle
        self.max_angle = max_angle
        self.min_pulse_width = min_pulse_width
        self.max_pulse_width = max_pulse_width
        self.set_angle(self.angle)
        

    def max(self):
        duty_ns_max = int(self.max_pulse_width*10**9)
        self.servo.duty_ns(duty_ns_max)
     
    def min(self):
        duty_ns_min = int(self.min_pulse_width*10**9)
        self.servo.duty_ns(duty_ns_min)

    def mid(self):
        mid_pulse_width = self.min_pulse_width + ((self.max_pulse_width - self.min_pulse_width)/2)
        duty_ns_mid = int(mid_pulse_width*10**9)
        self.servo.duty_ns(duty_ns_mid)

    def set_angle(self, angle):
        # Ausführung nur für erlaubte Winkel
        if angle >= self.min_angle and angle <= self.max_angle:
            self.angle = angle
            # Pulsweite für angle berechnen
            # zuerst Anteil am "Gesamtwinkel"
            angle_range = self.max_angle - self.min_angle
            proportion = (self.angle - self.min_angle) / angle_range

            # Anteil übertragen auf die Pulsweite
            pulse_range = self.max_pulse_width - self.min_pulse_width
            pulse_width_for_angle = pulse_range * proportion + self.min_pulse_width
            duty_ns = int(pulse_width_for_angle*10**9)
            self.servo.duty_ns(duty_ns)
        else:
            print("Angle not allowed.")
            
    def get_angle_for_duty_ns(self,duty_ns):
        # Umrechnen in Sekunden
        pulse_width = duty_ns/10**9
        pulse_range = self.max_pulse_width - self.min_pulse_width
        angle_range = self.max_angle - self.min_angle

        proportion = (pulse_width - self.min_pulse_width) / pulse_range
        angle_for_pulse_width = angle_range * proportion + self.min_angle
        
        return round(angle_for_pulse_width)            
        
    def set_angle_slow(self, angle, step_size=1, delay=.02):       
        # aktuelle Position ermitteln
        actual_duty_ns = self.servo.duty_ns()
        # Pulsweite in Grad umrechnen
        actual_angle = self.get_angle_for_duty_ns(actual_duty_ns)
        # Drehwinkel berechnen
        angle_diff = angle - actual_angle
        
        if angle_diff > 0: # drehe nach rechts
            while actual_angle < angle:
                self.set_angle(actual_angle)
                actual_angle = actual_angle + step_size
                time.sleep(delay)
            self.set_angle(angle)
        else:
            while actual_angle > angle:
                self.set_angle(actual_angle)
                actual_angle = actual_angle - step_size
                time.sleep(delay)
            self.set_angle(angle)
        
        # aktuelle Position ermitteln
        actual_duty_ns = self.servo.duty_ns()
        # in Grad umrechnen
        actual_angle = self.get_angle_for_duty_ns(actual_duty_ns)
```        

###






Ein Problem ist, dass die Motoren vor Beendigung des Programms wieder in ihre Startposition gebracht werden sollten, da es sonst beim Programmstart zu schnellen Bewegungen kommt. 

atexit




## Weiteres zum Thema

Roboterarm 0.1 Horizontaler Gelenkarmroboter:
https://www.youtube.com/watch?v=p1PwYa4Fdqc
* 5 Servos (3 drehen, 1 anheben, 1 greifen)
* Arduino Uno
* Stellgenauigkeit von Servos
* 3 Gelenke - 
* Probleme Ungenauigkeit, Schwingungen
* Kleiner Arm mit Potentiometer


https://www.youtube.com/watch?v=8uZkHoSezSc