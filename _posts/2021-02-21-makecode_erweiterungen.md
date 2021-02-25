---
layout: post
title:  "Erweiterungen für Makecode erstellen"
date:   2021-02-21
tags: Makecode Calliope Github L298
---


Bei der Programmierung des Calliope Mini mit Makecode möchte man vielleicht einige der selbst erstellten Funktionen in mehreren Projekten verwenden oder die Funktionen anderen zur Verfügung stellen. Hierfür kann man selbst Erweiterungen erstellen.

## Vorgehensweise

Die Vorgehensweise ist hier beschrieben: [https://makecode.calliope.cc/blocks/custom](https://makecode.calliope.cc/blocks/custom), wir führen folgende Schritte durch:

1. Neues Projekt in Makecode erstellen

2. Code nach Github laden ([siehe hierzu meinen Post zur Nutzung von Github in Makecode]({% post_url 2021-02-04-makecode_github %}))

3. Wechsel in die Javascript-Ansicht

Links erscheint dann im sog. Explorer eine Übersicht, der zum Projekt gehörenden Dateien. Für unsere Erweiterungen fügen wir durch Klick auf `+` eine weitere Datei hinzu. Diese nennen wir `custom.ts`. Klickt man nun links auf `custom.ts`, bekommt man rechts ein Fenster in dem Programmcode in [TypeScript](https://makecode.com/language/) erstellt und bearbeitet werden kann. 

In einer Erweiterung will man in der Regel eigene Blöcke definieren, die dann in Makecode verwendet werden können (siehe hierzu auch [https://makecode.com/defining-blocks](https://makecode.com/defining-blocks)). Im [Makecode Playground](https://makecode.com/playground) kann man das Erstellen von Blöcken ausprobieren und verschiedene Beispiele anschauen.

Funktionen werden über Makros mit Blocks verbunden. Die Makros werden in Kommentarzeilen geschrieben und beginnen innerhalb des Kommentars mit `%`. Die grundsätzliche Struktur sind so aus:

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

`namespace` ist der Bereich in der Menüleiste, in dem die Blöcke angezeigt werden. Mit `block` werden Funktionen gekennzeichnet, die als Block angezeigt werden. Die möglichen Optionen sind unter [https://makecode.com/defining-blocks](https://makecode.com/defining-blocks) beschrieben.

## Beispiel: Blöcke für Motor und Fahrzeug bei Nutzung eines L298-Motortreibers

Im Post [Motoren über den Motortreiber L298 am Calliope mini betreiben]({% post_url 2021-02-02-l298-calliope %}) habe ich  einige Funktionen zur Motor- und Fahrzeugsteuerung genutzt, die man in anderen Projekten gut brauchen könnte. Daher habe ich hierfür eine Erweiterung erstellt.

Dabei habe ich mich an der [Erweiterung für für Neopixel](https://github.com/microsoft/pxt-neopixel) orientiert und für die Erweiterung folgende Datei l298.ts erstellt:

```javascript
//% color="#AA278D" weight=8 icon="\uf1ba"
namespace l298 {

    export class L298Motor{
        in1: DigitalPin;
        in2: DigitalPin;
        en: AnalogPin;
        /**
        * Run the motor. 
        * @param speed a value between -100 and 100. speed < 0 is backwards
        **/
        //% block="turn motor $this(motor) with $speed"
        //% block.loc.de="drehe Motor $this(motor) mit Geschwindigkeit $speed"        
        //% speed.min=-100 speed.max=100
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
        /**
        * Stop the motor. 
        **/
        //% block="stop motor $this(motor)"
        //% block.loc.de="stoppe Motor $this(motor)"
        stop():void{
            this.turn(0)
        }
    }

    export class Vehicle{
        motor_left: L298Motor;
        motor_right: L298Motor;
        /**
        * Run the vehicle. 
        * @param speed a value between -100 and 100. speed < 0 is backwards
        * @param direction a value between -100 and 100. -100 is left (left motor stopped), 0 is straight, 100 is right (right motor stopped)
        **/
        //% speed.min=-100 speed.max=100
        //% direction.min=-100 direction.max=100
        //% block="drive vehicle $this(vehicle) with $speed and direction $direction"
        //% block.loc.de="fahre Fahrzeug $this(vehicle) mit Geschwindigkeit $speed und Richtung $direction"
        //% jsdoc.loc.de="Setzt das Fahrzeug in Fahrt."
        drive(speed: number, direction: number): void {
            if (direction <= 0) {
                this.motor_left.turn(speed * (100 + direction) / 100)
                this.motor_right.turn(speed)
            } else {
                this.motor_left.turn(speed)
                this.motor_right.turn(speed * (100 - direction) / 100)
            }
        }
        /**
        * Turn the vehicle left (left motor backwards, right motor forwards).
        * @param speed a value between 0 and 100 
        */
        //% block="turn vehicle $this(vehicle) to left with speed $speed"
        //% block.loc.de="drehe Fahrzeug $this(vehicle) mit Geschwindigkeit $speed nach links"
        //% speed.min=0 speed.max=100
        turn_left(speed:number):void{
            this.motor_left.turn(-speed)
            this.motor_right.turn(speed)
        }
        /**
        * Turn the vehicle right (left motor forwards, right motor backwards)
        * @param speed a value between 0 and 100
        */
        //% block="turn vehicle $this(vehicle) to right with speed $speed"
        //% block.loc.de="drehe Fahrzeug $this(vehicle) mit Geschwindigkeit $speed nach rechts"
        //% speed.min=0 speed.max=100
        turn_right(speed:number):void{
            this.motor_left.turn(speed)
            this.motor_right.turn(-speed)
        }
        /**
        * Stop the vehicle. 
        **/
        //% block="stop vehicle $this(vehicle)"
        //% block.loc.de="stoppe Fahrzeug $this(vehicle)"
        stop():void{
            this.motor_left.stop()
            this.motor_right.stop()
        }
    }

    /**
     * Create a new motor object for L298 motor driver.
     * @param in1 the pin where IN1/IN3 of L298 is connected.
     * @param in2 the pin where IN2/IN4 of L298 is connected.
     * @param en the pin where ENA/ENB of L298 is connected.
    **/
    //% blockId="l298_motor_create"
    //% block="Motor on IN1/IN3: %in1 IN2/IN4: %in2 EN: %en"
    //% block.loc.de="Motor an IN1/IN3: %in1 IN2/IN4: %in2 EN: %en"
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
    **/
    //% blockId="vehicle_create"
    //% block="Vehicle with motor left: %motor_left motor right: %motor_right"
    //% block.loc.de="Fahrzeug mit Motor links: %motor_left Motor rechts: %motor_right"   
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
```

Durch diesen Code wird zunächst ein neuer Bereich (namespace) namens L298 erzeugt, in dem die neuen Blöcke zu finden sind. Darin werden zwei Klassen, nämlich `L298Motor` und `Vehicle` definiert.

Die Klasse `L298Motor` hat als Instanzvariablen `in1`, `in2` und `en` die drei Pins, an denen der Motor angesschlossen ist. Sie hat eine Methode `turn` mit der Geschwindigkeit `speed` als Parameter, für die Werte zwischen -100 und 100 möglich sind. Bei Aufruf der Methode mit Werten größer 0 dreht sich der Motor vorwärts, bei Werten kleiner 0 rückwärts. Beim Aufruf mit 0 bleibt der Motor stehen. Innerhalb der Methode werden die für die jeweiligen Bewegungen erforderlichen Signale an den Pins erzeugt. Eine weitere Methode `stop` stoppt den Motor.

Die Klasse `Vehicle` hat die Instanzvariablen `motor_left` und `motor_right`, die auf Objekte der Klasse `L298Motor` verweisen. Sie hat die Methode `drive` mit den Parametern `speed` und `richtung`, die jeweils Werte zwischen -100 und 100 annehmen können. Ist `speed` größer 0 fährt das Fahrzeug vorwärts, bei `speed` kleiner 0 fährt es rückwärts, bei 0 bleibt es stehen. Bei der Richtung bedeutet -100 links, 0 geradeaus und 100 rechts. Innerhalb der Methode wird dann die `turn`-Methode auf den beiden Motoren aufgerufen. Die beiden weiteren Methoden `turn_left` und `turn_right` drehen das Fahrzeug jeweils auf der Stelle, indem ein Motor vorwärts und der andere rückwärts bewegt wird. Zudem gibt es eine Methode `stop` um das Fahrzeug zu stoppen.

Außerhalb der Klassendefinition befinden sich die Definiton der Funktionen `create_motor` und `create_vehicle`. Deren Aufgabe ist jeweils die Erzeugung der Objekte auf Basis der oben definierten Klassen. Dabei werden die Instanzvariable auf die jeweils übergebenen Objekte gesetzt.

Die mit `//%` gekennzeichneten Zeilen sind für die Erzeugung der Blöcke zuständig. An den erzeugten Blöcken sieht man am besten, was die einzelnen Einstellungen bewirken:

![Github in Makecode](/images/makecode_l298_erweiterung.png) 

Ein Teil der Übersetzungen (jsDoc und Parameter) wurden aus dem obigen Codebeispiel aus Gründen der Übersichtlichkeit entfernt. 


## Einbinden der Erweiterung

Die Erweiterung für den L298 könnt Ihr dadurch einbinden, dass Ihr in Makecode unter "Fortgeschritten" - "Erweiterungen" folgende Adresse eingebt:
 [https://github.com/verena-km/calliope_l298_motors](https://github.com/verena-km/calliope_l298_motors)

