## Funktionstest am ESP32

Den Speedsensor haben wir ja am Calliope bereits ausprobiert. Jetzt nutzen wir ihn am ESP32.

Ausgangspunkt ist unser Versuchsaufbau mit dem über den L298 angeschlossenen Motor am ESP32, wir benötigen also:

* eine Batteriebox
* einen Lego-Motor
* den Speedsensor LM393
* eine Lego-Technic-Riemenscheibe als Lochscheibe
* den ESP32

Wir stecken die Riemenscheibe auf eine mit dem Motor verbundene Lego-Achse und platzieren sie in der Lichtschranke des Sensors.

Der Sensor hat vier Ausgänge. Diese verbinden wir wie folgt:
* Pin VCC am Sensor mit VCC am ESP32
* Pin GND am Sensor mit GND am ESP32
* Pin D0 am Sensor mit Pin 13 am ESP32
* Pin A0 am Sensor bleibt ungenutzt

![Anschluss Speedsensor an ESP32](TODO: richtiges BIld/images/fritzing_calliope_anschluss_speedsensor.png)

Die Klasse Motor ([siehe hier](Link)) erweitern wir nun um die Möglichkeit, bei den Drehungen mit anzugeben, um wieviele Hell-Dunkel- bzw. Dunkel-Hell-Wechsel auf der Lochscheibe sich der Motor drehen soll:

```python
class EncoderMotor(Motor):
    def __init__(self, ena_nr, in1_nr, in2_nr, speedsens_nr):
        super().__init__(ena_nr, in1_nr, in2_nr)
        self.speedsens = Pin(speedsens_nr, Pin.IN, Pin.PULL_UP)
        self.speedsens.irq(trigger=Pin.IRQ_RISING|Pin.IRQ_FALLING, handler=self.callback)
        self.changes = 0

    def callback(self,pin):
        self.changes = self.changes+1

    def forward(self,speed=100, sections=None):
        super().forward(speed)
        if sections:
            self.changes=0
            while self.changes < sections:
                pass
            self.stop()

    def backward(self, speed=100,sections=None):
        super().backward(speed)
        if sections:
            self.changes=0
            while self.changes < sections:
                pass
            self.stop()

    def stop(self):
        self.in1.value(0)
        self.in2.value(0)
```

Der Konstruktor wird um einen weiteren Parameter ergänzt: Die Pin-Nummer an der der Speedsensor angeschlossen ist. Dieser wird als Eingang mit Pullup-Widerstand konfiguriert.

Die Lochscheibe erzeugt beim Drehen des Motors laufende Wechsel zwischen 0 und 1 an dem ensprechenden Pin des ESP. Bei diesen Wechseln in beide Richtungen, also von 0 auf 1 (IRQ-RISING) und von 1 auf 0 (IRQ-FALLING) soll jeweils ein Zähler hochgezählt werden. Wir definieren dazu einen Interrupt, der auf diese Wechsel reagiert und dann die Methode `callback` als Interrupt-Handler ausführt. Darin wird die Instanz-Variable `self.changes` hochgezählt.

Die Nutzung sieht dann so aus:

```python
motor1 = EncoderMotor(4,2,15,13)
motor1.forward(speed=50, sections=30)
```

Die Riemenscheibe hat 6 Löcher, also werden bei einer Umdrehung 12 Wechsel registriert. Ein Bewegen um 30 "sections" bedeutet damit 2,5 Umdrehungen.