




https://www.littlebird.com.au/a/how-to/98/pushbutton-with-micro-bit

```javascript
input.onPinPressed(TouchPin.P0, function () {
    pins.digitalWritePin(DigitalPin.P1, 1)
})
input.onPinReleased(TouchPin.P0, function () {
    pins.digitalWritePin(DigitalPin.P1, 0)
})
pins.setPull(DigitalPin.P0, PinPullMode.PullNone)
pins.setEvents(DigitalPin.P0, PinEventType.Edge)
```

Man hat für jeden Pin folgende Konfigurationsmöglichkeiten:
* den PinPullMode
* den PinEventType

Mit dem PinPullMode stellt man ein, ....

Mit dem PinEventType stellt man ein, welche Events für den Pin erzeugt werden. Dabei gibt es folgende Werte:
* Rand-Ereignisse (PinEventType.Edge)
* Puls-Ereignisse (PinEventType.Pulse)
* Berührungs-Ereignisse (PinEventType.Touch)
* keine Ereignisse (PinEventType.None)

Folgende vorgefertigten Eventhandler für Pins gibt es:
* pins.onPulsed
* input.onPinPressed
* input.onPinReleased



Test von Eventhandlern:

## onPinPressed und onPinReleased
* werden jeweils beim Öffnen einer Verbindung zwischen Pin und GND ausgeführt
* konfiguriert man beide, wird nur onPinReleased ausgeführt
* erfordert PinEventType Berührungs-Ereignisse (PinEventType.Touch) oder keine Angabe eines PinEventType
* wecher PinPullMode ist egal


## onPulsed
pins.onPulsed(DigitalPin.P1, PulseValue.High, () => {
    serial.writeValue("high", pins.pulseDuration())
})

* wird bei PulseValue.High jeweils ausgeführt beim Öffnen einer Verbindung zwischen Pin und VCC
-> Impulsdauer = die Dauer der Verbindung - also so lange es High war

pins.onPulsed(DigitalPin.P1, PulseValue.Low, () => {
    serial.writeValue("low", pins.pulseDuration())
})

* wird bei PulseValue.Low jeweis ausgeführt beim Aufbau einer Verbindung zwischen Pin und VCC
-> Impulsdauer = die Dauer, wie lange keine Verbindung war - also wie lange es Low war

Mit Hilfe beider onPulsed-EventHandler kann man somit Aktiviäten beim Wechsel von 0 nach 1 als auch von 1 nach 0 ausführen lassen und auch die Zeitdauer des jeweiligen Pulses nutzen.

onPulsed-Eventhandler:
* erfordert PinEventType Puls-Ereignisse (PinEventType.Pulse) oder keine Angabe eines PinEventType
* bei PinPullMode.PullDown werden die Wechsel zwischen Pin und VCC ausgewertet
* bei PinPullMode.PullUp werden die Wechsel zwichen Pin und Ground ausgewertet
