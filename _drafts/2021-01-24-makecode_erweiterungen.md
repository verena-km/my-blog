---
layout: post
title:  "Erweiterungen für Makecode erstellen"
date:   2021-01-04
tags: Makecode Calliope Github
---

Hier beschreibe ich, wie man eigene Erweiterungen für Makecode erstellt.

Bestimmte Funktionen möchte man vielleicht in mehreren Projekten verwenden oder man möchte diese anderen zur Verfügung stellen.

## Vorgehensweise

Die Vorgehensweise ist hier beschrieben: [https://makecode.calliope.cc/blocks/custom](https://makecode.calliope.cc/blocks/custom), wir führen folgende Schritte durch:

1. Neues Projekt in Makecode erstellen

2. Code nach Github laden (siehe hierzu ...)

3. Wechsel in die Javascript-Ansicht

Links erscheint dann im sog. Explorer eine Übersicht, der zum Projekt gehörenden Dateien. Für unsere Erweiterungen fügen wir durch Klick auf + eine weitere Datei hinzu. Diese nennen wir custom.ts.

Klickt man nun links auf custom.ts, bekommt man rechts ein Fenster in dem Programmcode in [TypeScript](https://makecode.com/language/) erstellt und bearbeitet werden kann. 

In einer Erweiterung will man in der Regel eigene Blöcke definieren, die dann in Makecode verwendet werden können (siehe hierzu auch [https://makecode.com/defining-blocks](https://makecode.com/defining-blocks))

Im [Playground](https://makecode.com/playground) kann man das Erstellen von Blöcken ausprobieren und verschiedene Beispiele anschauen.

Funktionen werden über Makros mit Blocks verbunden. Die Makros werden in Kommentarzeilen geschrieben und beginnen innerhalb des Kommentars mit `%`. Die grundsätzliche Struktur sind so aus

```javascript
//% color=#34c9eb weight=10
namespace bereichsname {
    //% block
    export function helloWorld() {
    }

    //% block
    export function camlCaseTwo() {
    }
}
```

`namespace` ist der Bereich in der Menüleiste, in dem die Blöcke angezeigt werden.

Für `namespace` sind folgende Makrooptionen möglich:
* color - Die Farbe als Hexadezimalwert, z.B. color=#34c9eb.
* weight - Die Position des Bereichs innerhalb der Menüleiste, je niedriger die Zahl, desto weiter unten, z.B. weight=10
* icon - ein Unicode-Zeichencode

Für die Funktionen  gibt es folgende Optionen 
* block - für die Funktion wird ein Block generiert

https://github.com/microsoft/pxt-neopixel/blob/master/neopixel.ts#L495



//% color="#AA278D" weight=8 icon="\uf1ba"
namespace l298 {

    export class L298Motor{
        in1: DigitalPin;
        in2: DigitalPin;
        en: AnalogPin;

        /**
        * Run the motor. 
        * @param speed a value between -100 and 100. speed < 0 is backwards
        */
        //% speed.min=-100 speed.max=100
        //% block="drehe Motor $this(motor) mit Geschwindigkeit $speed"
        turn(speed: number): void {   
            if (speed > 0) {
                pins.servoSetPulse(this.en, speed * 200)
                pins.digitalWritePin(this.in1, 0)
                pins.digitalWritePin(this.in2, 1)
            } else if (speed < 0) {
                pins.servoSetPulse(this.en, -speed * 200)
                pins.digitalWritePin(this.in2, 0)
                pins.digitalWritePin(this.in1, 1)
            } else {
                pins.analogWritePin(this.en, 0)
                basic.turnRgbLedOff()
            }
        }    
    }

    export class Vehicle{
        motor_left: L298Motor;
        motor_right: L298Motor;

        /**
        * Run the vehicle. 
        * @param speed a value between -100 and 100. speed < 0 is backwards
        * @param direction a value between -100 and 100. -100 is left, 0 is straight, 100 is right
        */
        //% speed.min=-100 speed.max=100
        //% richtung.min=-100 richtung.max=100
        //% block="fahre Fahrzeug $this(vehicle) mit Geschwindigkeit $speed und Richtung $richtung "
        fahre(speed: number, richtung: number): void {
            if (richtung <= 0) {
                this.motor_left.turn(speed * (100 + richtung) / 100)
                this.motor_right.turn(speed)
            } else {
                this.motor_left.turn(speed)
                this.motor_right.turn(speed * (100 - richtung) / 100)
            }
        }    
    }

    /**
     * Create a new motor object for L298 motor driver.
     * @param in1 the pin where IN1/IN3 of L298 is connected.
     * @param in2 the pin where IN2/IN4 of L298 is connected.
     * @param EN the pin where ena/enb of L298 is connected.
    */
    //% blockId="l298_motor_create" block="Motor an IN1/IN3: %in1 IN2/IN4: %in2 EN: %en"
    //% weight=90 blockGap=8
    //% blockSetVariable=motor
    //% trackArgs=0,2
    export function create_motor(in1: DigitalPin, in2:DigitalPin, en:AnalogPin): L298Motor {
        let motor = new L298Motor();
        motor.in1 = in1;
        motor.in2 = in2;
        motor.en = en;
        return motor;
    }
    /**
     * Create a new Vehicle object for two L298 motors.
     * @param motor_left the left motor
     * @param motor_right the right motor
    */
    //% blockId="vehicle_create" block="Vehicle mit Motor links: %motor_left Motor rechts: %motor_right"
    //% weight=90 blockGap=8
    //% blockSetVariable=vehicle
    //% trackArgs=0,2
    export function create_vehicle(motor_left: L298Motor, motor_right: L298Motor): Vehicle {
        let vehicle = new Vehicle();
        vehicle.motor_left = motor_left;
        vehicle.motor_right = motor_right;
        return vehicle;
    }
}