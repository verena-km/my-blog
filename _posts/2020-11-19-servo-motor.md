---
layout: post
title:  "Servomotor"
date:   2020-11-19
tags: Servomotor Raspberry ESP32 Calliope
---

Die Achse eines Servomotors kann man auf bestimmte Positionen in einem vorgegebenen Bereich (z.B. von 0 bis 90° oder von 0 bis 180°) drehen. Ein Servomotor ist daher für Drehbewegungen auf bestimmte Zielpositionen innerhalb des erlaubten Bereichs geeignet. Das ist beispielsweise bei einer Lenkung der Fall. Falls nötig, kann man auch mit einer Übersetzung (größeres Zahnrad am Servo treibt kleineres Zahnrad an) den Bereich vergrößern.

Einfache Servomotoren haben meist drei Anschlüsse:
* Rot für die Versorgungsspannung von 5 V
* Braun für Ground
* Orange für das Steuersignal

![Foto Servo-Motor](/images/foto_servo.jpg) 

Für die Ansteuerung der jeweiligen Winkelposition wird [Puls-Weiten-Modulation (PWM)]({% post_url 2020-11-16-pwm%}) genutzt und zwar meist ein Signal von 50 Hz, also ein Signal mit einer Periodenlänge von 20 ms. Die Position des Servos ist abhängig von der Dauer des High-Pegels im PWM-Signal (= Pulsweite / Duty Cycle). Dabei gilt in der Regel:

* 1,5 ms von 20 ms = 7,5 % entspricht 0° (Mittelposition)
* Pulsweite zwischen 0,5 und 2,5 ms, zum Teil auch weniger

Wenn die Pulsweite kürzer ist, dreht sich der Servo im Uhrzeigersinn, wenn er länger ist, dreht er sich gegen den Uhrzeigersinn. Bei manchen Servos ist es allerdings genau umgekehrt. Welcher Wert genau welchem Winkel entspricht, hängt vom jeweiligen Servo ab. Ebenso die Minimal und Maximalwerte. Die Werte sind im Datenblatt zufinden, manchmal weichen die tatsächlichen Werte aber von denen im Datenblatt ab. Dann hilft es, verschiedene Werte auszuprobieren und darüber festzustellen, welche Pulsweite welchem Winkel entspricht. Dies kann man beim ESP32 und beim Raspi in einer interaktiven Python-Session machen.

Für den oben abgebildeten Y-3009 gilt beispielsweise:

* 0,52 von 20 ms = Maximalausschlag im Uhrzeigersinn
* 2,33 von 20 ms = Maximalausschlag gegen den Uhrzeigersinn

* 0,56 von 20 ms = entspricht +90°
* 1,40 von 20 ms = entspricht 0°
* 2,33 von 20 ms = entspricht -90°

Für den MC-1811 gilt laut Datenblatt und auch nach meinen Messungen

* 1,5 von 20 ms = 0°
* 1,9 vom 20 ms = +45°
* 1,1 von 20 ms = -45°

## Ansteuerung unter Python mit dem Raspi

### Nutzung pigpio

Auf dem Raspi nutzen wir für ein stabiles PWM-Signal die Bibliothek pigpio. Einen Servo an Pin 18 kann man damit wie folgt ansteuern:

```python
import pigpio
import time

# der (lokale) Raspi
pi = pigpio.pi()

# Servo an Pin 18
servo_pin = 18
# als Ausgang konfigurieren
pi.set_mode(servo_pin, pigpio.OUTPUT)

# Unterschiedliche Positionen ansteuern
# pos ist die Pulsweite in µs, Wert zwischen 500 und 2500 
# 1500 ist normalerweise die Mitte 
for pos in [500, 1000, 1500, 2000, 2500, 1500]:
    pi.set_servo_pulsewidth(servo_pin,pos) 
    time.sleep(1)
# beenden
pi.set_servo_pulsewidth(servo_pin,0)
pi.stop()
```

### Nutzung gpiozero

Eine Alternative ist die Nutzung von gpiozero unter Verwendung des pigpiod.

In gpiozero gibt es die Klassen [Servo](https://gpiozero.readthedocs.io/en/stable/api_output.html#servo) und [AngularServo](
https://gpiozero.readthedocs.io/en/stable/api_output.html#servo).

Während Objekte der Klasses Servo nur auf Minimal-, - Maximal- und Mittelposition gebracht werden können, kann man bei AngularServo einen Winkel zur Positionierung angeben.

Beim Erzeugen des AngularServo-Objekts können folgende Parameter angegeben werden:
* pin - die Pinnummer
* inital_angle - der Startwinkel (default 0 = Mitte)
* min_angle - der Minimalwinkel (default -90)
* max_angle - der Maximalwinkel (default 90)
* min_pulse_width - die Pulsweite für den Minimalwinkel (default 1/1000)
* max_pulse_width - die Pulsweite für den Maximalwinkel (default 2/1000)
* pin_factory - als PinFactory nutzen wir ein pigpio um den Jitter zu vermeisen

Hier die Umsetzung für den MC-1811:

```python
from gpiozero import AngularServo
from gpiozero.pins.pigpio import PiGPIOFactory
from time import sleep

factory = PiGPIOFactory()
mc1811 = AngularServo(18, min_angle=-45,
                          max_angle=45,
                          min_pulse_width=1.1/1000,
                          max_pulse_width=1.9/1000, 
                          pin_factory = factory)
# auf Maximalposition gehen 
mc1811.max()
sleep(2)
# auf Minimalposition gehen
mc1811.min()
sleep(2)
# auf Mittelposition gehen
mc1811.mid()
sleep(2)
# auf Winkel -20° - ausgehend von der Mittelposition gehen
mc1811.angle=-20
```

Man kann auch für die Servotypen, die man häufig nutzt, eigene, von AngularServo abgeleitete Klassen erstellen:

```python
from gpiozero import AngularServo
from gpiozero.pins.pigpio import PiGPIOFactory
from time import sleep

factory = PiGPIOFactory()

class MC1811(AngularServo):
    def __init__(self, pin):
         super().__init__(pin,
                     min_angle=-45,
                     max_angle=45,
                     min_pulse_width=1.1/1000, 
                     max_pulse_width=1.9/1000,
                     pin_factory = factory)


class Y3009(AngularServo):
    def __init__(self, pin):
         super().__init__(pin,
                     min_angle=-90, 
                     max_angle=90, 
                     min_pulse_width=0.56/1000, 
                     max_pulse_width=2.33/1000, 
                     pin_factory = factory)
```

Diese kann man dann wie folgt nutzen:

```python
black_servo = MC1811(13)
black_servo.angle = (20)
```

## Ansteuerung unter Micropython mit dem ESP32

> Achtung!
> Beim ESP32 ist es mir nicht gelungen, den Servo über den Vin-Pin des mit 5 Volt zu versorgen. Der Aufruf der Duty-Methode führte dann zum Absturz. Mit einer externen Stromversorgung für den Servo klappt es. Zum Test habe ich das Lego-Batteriepack, den Step-Down-Konverter und den USB-Adapter (USB-A Stecker auf PS2-Buchse) mit herausgeführten Pins für 5 V und Ground ([siehe Stromversorgung]({% post_url 2020-09-12-stromversorgung%})) verwendet.
>
>Verbunden wurden dann: 
>* 5V und Ground aus der PS2-Buchse mit dem Servo
>* Ground aus der PS2-Buchse mit Ground vom ESP32
>* Pin 13 vom ESP mit dem Steuerleitung des Servos

Unter Micropython kann man wie folgt ein PWM-Signal an Pin 13 erzeugen. Für die erforderlichen Duty-Cycle Werte muss man etwas rechnen:

Die Mittelposition entspricht 1,5 ms von 20 ms = 7,5%. Die Werte für den Duty-Cycle liegen in Micropython zwischen 0 (off) und 1023 (on). Damit entspricht 7,5 % einem Wert von  7,5% * 1023 = 77. 

Für den MC-1811 gilt also:
* Maximal: 1,5 von 20 ms = 0° = 7,5% = 77
* Mitte: 1,9 vom 20 ms = +45° = 97 
* Minimal: 1,1 von 20 ms = -45° = 56

Die Ansteuerung für diese drei Werte sieht dann folgendermaßen aus.

``` python
from machine import Pin, PWM
import time
servo = PWM(Pin(13,Pin.OUT))
servo.freq(50)
servo.duty(77)
time.sleep(1)
servo.duty(97)
time.sleep(1)
servo.duty(56)
```

In MicroPython kann man ebenfalls Klassen für die Servos definieren, was die spätere Nutzung vereinfacht. Wir orientieren uns dabei an der Klasse AngluarSevero aus der gpiozero-Biblothek für den Raspi (siehe oben): 

``` python
from  machine import Pin, PWM
import time

# Allgemeine Klasse für Servos mit Winkelansteuerung
class AngularServo:
    def __init__(self, pin,
                          angle = 0,
                          min_angle = -90,
                          max_angle = 90,
                          min_pulse_width = 1.0/1000,
                          max_pulse_width = 2.0/1000):
        self.servo = PWM(Pin(pin,Pin.OUT))
        self.servo.freq(50)
        self.angle = angle
        self.min_angle = min_angle
        self.max_angle = max_angle
        self.min_pulse_width = min_pulse_width
        self.max_pulse_width = max_pulse_width
        self.set_angle(self.angle)

    def calculate_dutycycle(self, pulse_width):
        # pulse_width is in ms - calculate percent from period 20 ms (50Hz)
        percent = pulse_width / (20.0/1000)
        # micropython PWM needs int values between 0 (off) an 1023 (on)
        dutycycle = int(1023 * percent)
        return dutycycle

    def max(self):
        self.servo.duty(self.calculate_dutycycle(self.max_pulse_width))
     
    def min(self):
        self.servo.duty(self.calculate_dutycycle(self.min_pulse_width))

    def mid(self):
        mid_pulse_width = self.min_pulse_width + ((self.max_pulse_width - self.min_pulse_width)/2)
        self.servo.duty(self.calculate_dutycycle(mid_pulse_width))

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

            # Umrechnen in dutycycle
            dutycycle = self.calculate_dutycycle(pulse_width_for_angle)
            self.servo.duty(dutycycle)
        else:
            print("Angle not allowed.")

# Abgeleitete Klassen für zwei bestimmte Servotypen
class MC1811(AngularServo):
    def __init__(self, pin):
         super().__init__(pin, min_angle=-45, max_angle=45, min_pulse_width=1.1/1000, max_pulse_width=1.9/1000)

class Y3009(AngularServo):
    def __init__(self, pin):
         super().__init__(pin, min_angle=-90, max_angle=90, min_pulse_width=0.56/1000, max_pulse_width=2.33/1000)

# Nutzungsbeispiel
if __name__ == "__main__":
    black_servo = MC1811(13)
    black_servo.min()
    time.sleep(2)
    black_servo.max()
    time.sleep(2)
    black_servo.mid()
    time.sleep(2)
    black_servo.set_angle(20)
    time.sleep(2)
    black_servo.set_angle(50)
``` 

## Ansteuerung mit dem Calliope

Auch beim Calliope benötigt man eine externe Stromversorgung für den Servo. Ich verwende wiederum das Lego-Batteriepack, den Step-Down-Konverter und den USB-Adapter (USB-A Stecker auf PS2-Buchse) mit herausgeführten Pins für 5 V und Ground ([siehe Stromversorgung]({% post_url 2020-09-12-stromversorgung%})).

Der Anschluss an den Calliope geht dann wie folgt:
 
* 5V mit der PS2-Buchse mit der roten Leitung des Servo
* Ground aus der PS2-Buchse auf ein Breadbord
* Ground vom Breadboard mit der braunen Leitung des Servo
* Ground vom Breadboard mit Ground vom Calliope (z.B. mit Krokodilklemme an die "Minuspol-Ecke")
* die gelbe Steuerleitung des Servos mit einem PWM-Ausgang des Calliope, z.B. mit der Ecke P1 links unten

Ein einfaches Programm in Makecode sieht dann folgendermaßen aus:

```javascript
input.onButtonPressed(Button.A, function () {
    pins.servoSetPulse(AnalogPin.P1, 1100)
})
input.onButtonPressed(Button.AB, function () {
    pins.servoSetPulse(AnalogPin.P1, 1500)
})
input.onButtonPressed(Button.B, function () {
    pins.servoSetPulse(AnalogPin.P1, 1900)
})
pins.servoSetPulse(AnalogPin.P1, 1500)
```

Abhängig vom gedrückten Knopf (A, B oder beide) werden unterschiedliche Servo-Positonen angefahren. Für die Positon muss man die Pulsweite angeben, und zwar in Microsekunden. Die Mittelposition entspricht meist dem Wert 1500.