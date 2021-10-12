---
layout: post
title:  "Digitale Eingänge"
date:   2021-10-12
tags: GPIO Pullup Pulldown Callback Raspberry ESP32 Calliope
---

Verwendet man GPIO-Pins als digitale Eingänge, kann man z.B. den Zustand eines Schalters oder Tasters abfragen. Um dabei für einen definierten Grundzustand zu sorgen, baut man einen Widerstand ein.

Je nach dem, wie man den Schalter mit dem GPIO-Pin und der Spannungsversorgung sowie dem Widerstand verbindet, nennt man den Widerstand Pullup- bzw. Pulldown-Widerstand.

Man kann die Widerstände extern z.B. auf einem Breadboard einbauen, oder man nutzt Widerstände, die auf dem Microcontroller über einen Programmbefehl zugeschaltet werden können.

## Hardware-Widerstände einbauen

Die Widerstände werden wie folgt eingebaut:

Pullup:
* Beim Pullup wird der GPIO-Pin über einen Widerstand mit der Betriebsspannung verbunden. 
* Der Schalter wird auf der einen Seite mit dem GPIO-Pin und auf der anderen Seite mit GND verbunden.

Pulldown:
* Beim Pulldown wird der GPIO-Pin über einen Widerstand mit GND verbunden.
* Der Schalter wird auf der einen Seite mit dem GPIO-Pinn und auf der anderen Seite mit der Betriebsspannung verbunden.

Empfohlen ist ein Widerstand mit 10 kOhm.

Nachfolgendes Bild zeigt links den Pullup- und rechts den Pulldown-Widerstand am Beispiel des Calliope Mini.

![Pullup- und Pulldown-Widerstand](/images/fritzing_pullup_pulldown.png)

Beim Pullup-Widerstand (Bild links) gilt:
* bei offenem Schalter: GPIO-Pin auf HIGH (1)
* bei geschlossenem Schalter: GPIO-Pin auf LOW (0)

Beim Pulldown-Widerstand (Bild rechts) gilt:
* bei offenem Schalter: GPIO-Pin auf LOW (0)
* bei geschlossenem Schalter: GPIO auf HIGH (1)

Detailliertere Erläuterungen zum Thema Pullup- und Pulldown-Widerstände findet man im [Elektronik-Kompendium](https://www.elektronik-kompendium.de/sites/raspberry-pi/2006051.htm).

## Softwareseitig zuschaltbare Widerstände nutzen

Beim Raspi, beim ESP32 und beim Calliope gibt es auch softwareseitig zuschaltbare Pullup- und Pulldown-Widerstände. Auf der sicheren Seite ist man aber bei der Verwendung eines "echten Widerstandes".

Nutzt man die softwareseitig zuschaltbaren Widerstände sieht der Schaltplan wie folgt aus (links Pullup, rechts Pulldown): 

![Pullup- und Pulldown-Widerstand](/images/fritzing_internal_pullup_pulldown.png)

Hier gilt ebenfalls:

Pullup (Bild links):
* bei offenem Schalter: GPIO-Pin auf HIGH (1)
* bei geschlossenem Schalter: GPIO-Pin auf LOW (0)

Pulldown (Bild rechts):
* bei offenem Schalter: GPIO-Pin auf LOW (0)
* bei geschlossenem Schalter: GPIO auf HIGH (1)

Nachfolgend ist beschrieben, wie die internen Widerstände aktiviert werden können.

### MicroPython / ESP32

In folgendem Beispiel für den ESP32 mit MicroPython wird Pin 23 mit einem Pullup-Widerstand konfiguriert.

```python
from machine import Pin
import time

taster = Pin(23, Pin.IN, Pin.PULL_UP)

while True:
    print(taster.value())
    time.sleep_ms(50)
```

Bei offenem Schalter bzw. nicht gedrücktem Taster wird eine 1 ausgegeben, bei geschlossenem Schalter bzw. gedrücktem Taster eine 0.

Nutzt man einen Pulldown-Widerstand, sieht das Beispiel wie folgt aus:

```python
from machine import Pin
import time

taster = Pin(23, Pin.IN, Pin.PULL_DOWN)

while True:
    print(taster.value())
    time.sleep_ms(50)
```
Nun wird bei offenen Schalter bzw. nicht gedrücktem Taster eine 0 ausgegeben, bei geschlossenem Schalter bzw. gedrücktem Tater eine 1.

### Makecode / Calliope

Beim Calliope kann man in Makecode die Widerstände softwareseitig mit 

```javascript
pins.setPull(DigitalPin.P0, PinPullMode.PullUp)
```
bzw.

```javascript
pins.setPull(DigitalPin.P0, PinPullMode.PullDown)
```

aktivieren bzw. mit 
```javascript
pins.setPull(DigitalPin.P0, PinPullMode.PullNone)
```

Den entsprechenden Befehl in der Blockansicht findet man unter "Fortgeschritten" - "Pins" - "mehr". Die deutsche Übersetzung heisst merkwürdig: 

"setze Anziehungskraft von Pin P0 auf nach oben / nach unten / keine"

Werte an einem digitalen Eingang können mit `digitalReadPin` ausgelesen werden:

```javascript
basic.forever(function () {
    serial.writeNumber(pins.digitalReadPin(DigitalPin.P0))
    basic.pause(500)
})
```

### Python / RasperryPi

Beim Raspi kommt es darauf an, welche Bibliothek man verwendet:

Bei `RPi.GPIO` gibt man beim Setup des GPIO-Pins den zusätzlichen Parameter `pull_up_down` mit dem Wert `GPIO.PUD_DOWN` bzw. `GIPIO.PUD_UP` an:

```python
import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BCM)
pin = 4
GPIO.setup(pin, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)

while True:
    print(GPIO.input(pin))
```

```python
import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BCM)
pin = 4
GPIO.setup(pin, GPIO.IN, pull_up_down=GPIO.PUD_UP)

while True:
    print(GPIO.input(pin))
```

Nutzt man bei `gpiozero` die Klasse `Button` kann man bei Erzeugen des Objekts den Parameter `pull_up` angeben. Hat er den Wert `True` wird ein Pullup-Widerstand genutzt, bei `False` ein Pulldown-Widerstand, beim Wert `None` wird kein Widerstand verwendet. Als Default wird `True` verwendet.

```python
from gpiozero import Button
button = Button(4,pull_up=False)
while True:
    print(button.value)
```

## Aktionen beim Schließen bzw. Öffnen des Schalters oder Tasters

Will man beim Schließen bzw. Öffnen des Schalters oder Tasters bestimmte Aktionen ausführen - beispielsweise eine LED an- und ausschalten hat man folgende Möglichkeiten.

1. Man fragt in eine Schleife in kurzen Zeitabständen den am Eingang liegenden Wert ab, und prüft, ob dieser sich geändert hat. Ist dies der Fall, kann man bestimmte Aktionen ausführen.

2. Man nutzt die Möglichkeiten des jeweiligen Systems um für bestimmte Ereignisse - wie das Schließen des Schalters bestimmte Funktionen zu definieren, die bei Eintreten des Ereignisses ausgeführt werden (sog. Callback-Funktionen). 

Bei der Nutzung von Callback-Funktionen kann das Problem auftreten, dass bei Betätigen des Schalters kurzzeitig ein mehrfaches Schließen und öffnen des Kontakts auftritt, das sog. [Prellen (Bouncing)](https://www.mikrocontroller.net/articles/Entprellung). Dadurch kommt es auch zu mehreren Events und die Callback-Funktion wird unbeabsichtigt mehrfach ausgeführt.

Dem Problem kann man durch eine Entprellschaltung oder aber auch durch eine Entprellroutine in Software begegnen.


### Python / RasperryPi

Beim Raspberry-Pi werden für die Callback-Funktionen automatisch eigene Threads gestartet. 

Mit `RPI.GPIO` richtet man Callback-Funktionen wie in folgendem Beispiel ein:

```python
import RPi.GPIO as GPIO
import time
from signal import pause

GPIO.setmode(GPIO.BCM)
pin = 4
GPIO.setup(pin, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)

def say_hello(pin):
    print("Hello!")
   
GPIO.add_event_detect(pin, GPIO.RISING, callback=say_hello)
pause()
```
Dabei gibt es die folgenden Events:
* GPIO.RISING
* GPIO.FALLING
* GPIO.BOTH

Auf jedem Pin kann nur auf eines dieser Events ein Callback registriert werden.

Gegen das Prellen kann man einen Bouncetime-Wert in Millisekunden angeben:
```python
GPIO.add_event_detect(pin, GPIO.RISING, callback=say_hello, bouncetime=500)
```

Bei `gpiozero` sieht man die Nutzung an folgendem Beispiel:

```python
from gpiozero import Button
from signal import pause

def say_hello():
    print("Hello!")

def say_bye():
    print("ByeBye!")    

button = Button(4,pull_up=False)
button.when_pressed = say_hello
button.when_released = say_bye
pause()
```
Die Funktion say_hello wird bei jedem Wechsel von 0 auf 1 ausgeführt, die Funktion say_bye bei jedem Wechsel von 1 auf 0.

Auch hier kann man gegen das Prellen einen Wert (in Sekunden) für die Bouncetime angeben:
```python
button = Button(4,pull_up=False, bounce_time=0.5)
```

### MicroPython / ESP 32

In MicroPython kann man Pins so konfigurieren, dass bei Veränderung ihrer Input-Werte ein Interrupt ausgelöst wird:

```python
from machine import Pin

def callback(p):
    print('pin change', p)

p0 = Pin(0, Pin.IN)
p0.irq(trigger=Pin.IRQ_FALLING, handler=callback)
```

Mit der Methode `irq()` der Klasse Pin legt man einen solchen Interrupt fest und definiert mit `trigger`, ob er bei beim Wechsel von 0 nach 1 (Pin.IRQ_RISING) beim Wechsel von 1 nach 0 (Pin.IRQ_FALLING) oder bei beidem (Pin.IRQ_FALLING\|Pin.IRQ_RISING) ausgeführt werden soll. Als Parameter `handler` wird die Callback-Funkton angegeben, die ausgeführt werden soll, wenn das entsprechende Ereignis stattfindet. Die Callback-Funktion hat als einigen Parameter den Pin, der die Funktion ausgelöst hat.

Für die Interrupt-Handler gibt es einiges zu beachten, siehe hierzu die [Micro-Python-Dokumentation](https://docs.micropython.org/en/latest/reference/isr_rules.html). So sollten die Callback-Funktionen keine zeitaufwändigen Dinge tun. Auch ein `print`-Statement (wie oben) ist eingentlich zu langsam.

Was nicht geht, ist für RISING und FALLING getrennte IRQs zu definiert. Es wird nur einer ausgeführt.

Um Debouncing in Software muss man sich in MicroPython selbst kümmern.

Hier eine Lösung, bei der eine bestimmte Anzahl gleicher aufeinander folgender Werte erreicht sein muss und erst dann die eigentliche Callback-Funktion ausgeführt wird:
[https://micronote.tech/2020/02/Timers-and-Interrupts-with-a-NodeMCU-and-MicroPython/](https://micronote.tech/2020/02/Timers-and-Interrupts-with-a-NodeMCU-and-MicroPython/])

Hier eine Lösung mit einem Timer:
[https://kaspars.net/blog/micropython-button-debounce](https://kaspars.net/blog/micropython-button-debounce)


### Calliope

Auch beim Callipe kann man bestimmte Aktionen starten, wenn sich die Werte an den Eingangspins ändern. 

In Makecode kann man mit dem PinEventType einstellen, welche Events für den Pin erzeugt werden. Dabei gibt es folgende Werte:
* Rand-Ereignisse (PinEventType.Edge)
* Puls-Ereignisse (PinEventType.Pulse)
* Berührungs-Ereignisse (PinEventType.Touch)
* keine Ereignisse (PinEventType.None)

Gesetzt wird der PinEventType wie folgt:

```javascript
pins.setEvents(DigitalPin.P0, PinEventType.Touch)
```
Eigentlich muss man den PinEventType nicht setzen, das passiert wohl durch die Nutzung der einzelnen Events automatisch.

Beim Calliope (Makecode) gibt es folgende Events für digitale Eingänge:

* input.onPinPressed
* input.onPinReleased
* pins.onPulsed


#### onPinPressed und onPinReleased
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

#### onPulsed

Bei onPulsed kann man angeben, ob man Event beim Wechsel von High nach Low oder von Low nach High informiert werden will. 

```javascript
pins.onPulsed(DigitalPin.P0, PulseValue.High, function () {
    serial.writeValue("high", pins.pulseDuration())
})
pins.onPulsed(DigitalPin.P0, PulseValue.Low, function () {
    serial.writeValue("low", pins.pulseDuration())
})
```

Beim PinPullMode.PullDown gilt Folgendes:
* Es werden die Wechsel zwischen Pin und VCC ausgewertet.

* Bei PulseValue.High wird die Funktion jeweils ausgeführt beim Trennen einer Verbindung zwischen Pin und VCC. Dabei entspricht die Impulsdauer dem Zeitraum in dem die Verbindung bestand - also so lange es High war.

* Bei PulseValue.Low wird die Funktion jeweils ausgeführt beim Herstellen einer Verbindung zwischen Pin und VCC. Dabei entspricht die Impulsdauer dem Zeitraum, in dem keine Verbindung bestand - also wie lange es Low war.

Beim PinPullMode.PullUp gilt Folgendes:

* Es werden die Wechsel zwischen Pin und Ground ausgewertet.

* Bei PulseValue.High wird die Funktion jeweils ausgeführt beim Herstellen einer Verbindung zwischen Pin und Ground. Dabei entspricht die Impulsdauer dem Zeitraum in dem keine Verbindung bestand - also so lange es High war.

* Bei PulseValue.Low wird die Funktion jeweils ausgeführt beim Trennen einer Verbindung zwischen Pin und Ground. Dabei entspricht die Impulsdauer dem Zeitraum, in dem eine Verbindung bestand - also wie lange es Low war.

Mit Hilfe beider onPulsed-EventHandler kann man somit Aktiviäten beim Wechsel von 0 nach 1 als auch von 1 nach 0 ausführen lassen und auch die Zeitdauer des jeweiligen Pulses nutzen.