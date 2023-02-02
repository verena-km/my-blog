---
layout: post
title:  "Dynamixel Motoren"
date:   2022-11-06
tags: Motor Dynamixel
---

Dynamixel sind Servo-Motoren der Firma Robotis, die besonders für Robotik-Projekte geeignet sind. Mehrere Dynamixel-Motoren können in Reihe geschaltet werden (sog. Daisy Chaining). Jeder der Motoren kann dann über eine eindeutige ID angesprochen werden. Die Kommunikation läuft über die serielle Schnittstelle UART TTL.

Es gibt folgende Möglichkeiten, die Motoren zu bewegen:
* dauerhafte Drehung in einer bestimmten Geschwindigkeit (Velocity Control Mode)
* Drehung zu einer bestimmten Position - ggf auch mit mehreren Umdrehungen ((Extended) Position Control Mode)

Angesteuert werden können die Dynamixel-Motoren mit einem PC oder einem Microcontroller. 

Für die Ansteuerung mit einem PC kann man den U2D2-Adapter von Robotis verwenden. Für die Ansteuerung mit Microcontrollern gibt es mehrere Möglichkeiten:
- mit speziellen Shields (z.B. Dynamixel Shield für Arduino)
- über einen 
- direkt

Ich verwende [Dynamixel vom Typ XL430-W250](https://emanual.robotis.com/docs/en/dxl/x/xl430-w250/).

Diese benötigen Eine Spannungsversorgung zwischen 6,5 un d12 Volt.

Die Dynamixel-Motoren wird über das Senden und Empfangen von Datenpaketen über die serielle Schnittstelle programmiert. Das ist grundsätzlich mit allen Programmiersprachen möglich, das [Dynamixel SDK](https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_sdk/overview/) enthält Basisfunktionen und Beispiele für folgende Programmiersprachen:
* C, C++, C#
* Python
* Java
* MATLAB
* LAbView
* ROS


## Kennenlernen der Motoren mit dem Dynamixel-Wizard

Mit Hilfe des Dynamixel-Wizards kann man sich mit dem Dynamixel-Motoren und deren Ansteuerung vertraut machen. Die Software kann man hier herunterladen: [https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_wizard2/](https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_wizard2/)

Unter Linux muss man die Datei zunächst noch ausführbar machen

```
$ chmod u+x ./DynamixelWizard2Setup_x64
```

und kann dann das Setup starten:

```
$ ./DynamixelWizard2Setup_x64
```
Es erscheint ein Einrichtungsassistent, bei dem man Installationsordner auswählen muss. Nach der Installation findet man unter Ubuntu einen Menüeintrag `Dynamixel Wizard 2` der die Datei DynamixelWizard2.sh aus dem Installationsordner startet.

Mit Klicken auf Scan kann man prüfen, welche Motoren angeschlossen sind, sie werden dann links angezeigt. Unter Options - Scan kann man den zu scannenden Bereich auf bestimmte Baudraten eingschränken.

![Screenshot Dynamixel Wizzard](screenshot_dynamixel_wizard.png)

In der Mitte sieht man dann die "ControlTable". Das sind Speicherbereiche im EEPROM und im RAM des jeweiligen Motors. Klickt man diese an erhält man rechts unten entsprechende Details und kann diese dort ggf. ändern. Änderungen im EEPROM überdauern das Ausschalten, Änderungen im RAM sind flüchtig.

Rechts oben wird eine Steuerung angzeigt. In diese kann man
* den Motor auf Werkseinstellungen zurücksetzen (Factory Reset)
* einen Reboot durchführen
* den Betriebsmodus (Operating Mode) auswählen (Velocity, Position, Extended Position, PWM)
* die Drehung (Torque) an- oder aus schalten
* die LED an- oder ausschalten

Abhängig vom Betriebsmodus wird ein Bedienelement angezeigt, in dem man die anzusteuernde Position (bei Position und Extended Position) oder die Geschwindigkeit wählen (bei Velocity und PWM) kann.

Die Veränderung einzelner Werte kann man sich auch in einem Graphen anzeigen lassen. Im Packet-Fenster kann man manuell Datenpakete erstellen, an den Dynamixel senden und dessen Antwortpakete anschauen. So mit kann die Kommunikation über Datenpakete (siehe unten) ausprobieren.

Weitere Informationen sind hier zu finden:
* [im Manual](https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_wizard2/)
* in folgender https://www.daslhub.org/unlv/courses/me729/lesson-B-dynamixelWizard/lab/labXl320-introToDynamixelWizard-121419a.pdf


https://www.daslhub.org/unlv/courses/me729/lesson-B-dynamixelWizard/lab/labXl320-introToDynamixelWizard-121419a.pdf

https://www.youtube.com/watch?v=JRRZW_l1V-U

https://www.youtube.com/watch?v=cQIDjrfDb24

## RoboPlusManager


## Kommunikation über Datenpakete

Der Dynamixel wird über das Senden und Empfangen von Datenpaketen über die serielle Schnittstelle gesteuert. Für den XL430-W250 verwenden wir das [Protocol 2.0](https://emanual.robotis.com/docs/en/dxl/protocol2/).

In der Protokollbeschreibung findet man den Aufbau der Pakete, unterschieden wird zwischen
* dem "Instruction Packet" das an das Gerät gesendet wird
* dem "Status Packet" das als Antwort zurück übermittelt wird.

Folgende "Instructions" können übermittelt werden:
* Ping
    Prüft Erreichbarkeit des jeweiligen Motors.
    Es kommt ein Statuspaket mit Modellnummer und Firmwareversion zurück.
* Read
    Liest Werte aus der "Control Table".
    Es kommt ein Statuspaket mit dem angefragten Wert zurück.
* Write
    Schreibt Werte in die "Control Table".
    Es kommt ein Statuspaket zurück.
* RegWrite
    Die zu schreibenden Werte werden registriert, aber erst beim Empfang eines Action-Pakets ausgeführt (verbesserte Synchronisation)
    Es kommt ein Statuspaket zurück.    
* Action
    Ausführung zuvor registrierter Werte.
    Es kommt ein Statuspaket zurück.
* Factory Reset
    Zurücksetzen auf Werkseinstellungen.
    Es kommt ein Statuspaket zurück.
* Reboot
    Neustart des Motors
    Es kommt ein Statuspaket zurück.
* Clear
    Zurücksetzen bestimmter Register (Present Position)
    Es kommt ein Statuspaket zurück.
* Control Table Backup
    Sicherung des aktuellen Status des Control Table in einen Backup-Bereich
    Es kommt ein Statuspaket zurück.
* Sync Read
    Gleichzeitiges Lesen von Werten aus der "Control Table" mehrerer Motoren
    Adresse und Länge müssen gleich sein
    Es kommen mehrere Statuspakete zurück.
* Sync Write
    Gleichzeitiges Schreiben von Werten in die "Control Table" mehrerer Motoren
    Adresse und Länge müssen gleich sein
    Es kommt kein Statuspaket zurück.
* Fast Sync Read
    Gleichzeitiges Lesen von Werten aus der "Control Table" mehrerer Motoren
    Adresse und Länge müssen gleich sein
    Es kommt ein Statuspaket zurück.
* Bulk Read
    Gleichzeitiges Lesen von Werten in aus der "Control Table" mehrerer Motoren
    Auch unterschiedliche Adressen und Längen möglich
    Es kommen mehrere Statuspakete zurück.
* Bulk Write
    Gleichzeitiges Schreiben von Werten in die "Control Table" mehrerer Motoren
    Auch unterschiedliche Adressen und Längen möglich.
    Es kommt kein Statuspaket zurück.    
* Fast Sync Read
    Gleichzeitiges Lesen von Werten aus der "Control Table" mehrerer Motoren
    Adresse und Länge müssen gleich sein
    Es kommt ein Statuspakete zurück.
* Fast Bulk Read
    Gleichzeitiges Schreiben von Werten in die "Control Table" mehrerer Motoren
    Adresse und Länge müssen gleich sein
    Es kommt ein Statuspakete zurück.

## Aufbau der Datenpakete

Der grundsätzlich Aufbau der Pakete ist ähnlich, die Pakete können aber unterschiedlich lang sein, da die Anzahl der Parameter variieren kann. Nachfolgend eine kurze Übersicht über den Aufbau der Pakete:

Instruction-Pakete:
* 4 Byte Header immer gleich
* 1 Byte ID
* 2 Byte Länge (Anzahl der noch nachfolgenden Bytes)
* 1 Byte Instruction (eine der oben aufgelisteten)
* Parameter (variable Länge)
* 2 Byte Checksumme

Status-Pakete:
* 4 Byte Header immer gleich
* 1 Byte ID
* 2 Byte Länge (Anzahl der noch nachfolgenden Bytes)
* 1 Byte Instruction ( 0x55 für Status)
* 1 Byte Error
* Parameter (variable Länge)
* 2 Byte Checksumme

Abhängig von der Instruction ist die Länge der Parameter unterschiedlich. Hier die entsprechenden Angaben für die wichtigsten Instructions: 

Ping-Instruction:
* 0 Byte Parameter

Ping-Status:
* 3 Byte Parameter

Read-Instruction:
* 4 Byte Parameter, davon zwei für die Startadresse und zwei für die Länge der zu lesenden Daten in Byte

Read-Status:
* x Byte Parameter (entsprechend der Länge der Read-Instruction)

Man kann also mehrere aufeinanderfolgende Werte mit einer Read-Instruction lesen. Mit einer Read-Instruction ab Adresse 0 und einer Länge von 148 Bytes kann man die Control Table komplett lesen.

Write-Instruction:
* x Byte Parameter (2 Byte für die Startadresse und n Bytes für die zu schreibenden Werte)

Man kann also mehrere aufeinanderfolgende Werte mit einer Write-Instruction schreiben.

Write-Status:
* 0 Byte Parameter

## Control-Table

Die Control-Table ist abhängig vom jeweiligen Dynamixel-Modell. Sie Besteht aus dem EEPROM-Bereich und dem RAM-Bereich. In der Tabelle ist zu finden:
- welche Werte in der Control-Table zu finden sind
- an welcher Adresse sich diese befinden
- wie viele Bytes für diesen Wert verwendet werden
- ob der Wert nur gelesen oder auch verändert werden kann
- welcher Default-Wert verwendet wird
- welchen Wertebereich der Wert hat
- in welcher Einheit der Wert ist.

Für den XL430-W250 ist die Control Table hier zu finden:
(https://emanual.robotis.com/docs/en/dxl/x/xl430-w250/#control-table-of-eeprom-area)[https://emanual.robotis.com/docs/en/dxl/x/xl430-w250/#control-table-of-eeprom-area]


## Operating Modes

Es gibt (beim XL430-W250) vier Betriebsmodi, die  durch den Wert an Adresse 11 bestimmt werden:
* Velocity Control (Wert 1): Dauerhafte Drehbewegung
* Position (Wert 3, Default): Gezieltes Ansteuern von bestimmten Positionen
* Extended Position (Wert 4): Gezieltes Ansteuern von bestimmten Positionen mit mehrfachen Drehungen
* PWM (Wert 16): Dauerhafte Drehbewegung (?)

### Velocity Control Mode

Im Velocity Control Mode dreht sich der Motor dauerhaft. Die Geschwindigkeit wird durch den Wert "Goal Velocity" festgelegt, wobei der Wert "Velocity Limit" nicht überschritten werden darf. Der Wert 1 entspicht 0,229 Umdrehungen pro Minute. Als "Velocity Limit" kann man maximal 1023 angeben - also 234 Umdrehungen pro Minute. Erreicht werden aber maximal 280 - also 64 Umdrehungen pro Minute. Durch negative Werte erhält man die geänderte Drehrichtung.

### Postition Control Mode

Im Position Control Mode dreht sich der Motor hin zur angegebenen Position "Goal Position". Der Bereich der ansteuerbaren Positionen kann durch die Werte "Max Position Limit" und  "Min Position Limit" begrenzt werden. Maximal können die Positionen 0 bis 4095 angesteuert werden. Dies entspricht einer Umdrehung.

Die Drehrichtung geht dabei standardmäßig bei aufsteigenden Werten gegen den Uhrzeigersinn, bei absteigenden Werten im Uhrzeigersinn. Durch den Wert "Homing Offset" kann der Start- bzw. Mittelpunkt der Drehbewegung verschoben werden. Mit einem negativen Wert gegen den Uhrzeigersinn, mit einem positiven Wert im Uhrzeigersinn verschoben - maximal aber um jeweils 1024 Positionen. 

### Extended Position Control Mode
In diese Modus kann ebenfalls eine genaue Position "Goal Position" angesteuert werden, es sind aber auch mehrere (bis zu 512) Umdrehungen in jede Richtung möglich. Der Wertebereich für die "Goal Position" liegt zwischen -1.048.575 und 1.048.575. Die Drehrichtung geht hier standardmäßig ebenfalls bei aufsteigenden Werten gegen den Uhrzeigersinn, bei absteigenden Werten im Uhrzeigersinn.

## Profile
Durch die Verwendung von sog. Profilen können Probleme durch schnelle Beschleunigungen und Bremsungen - wie z.B. Vibrationen abgemildert werden. Es gibt folgende Profile: 

- sofort auf Zielposition (Step Instruction)
- Rechteck-Profil (Rectangular Profile) - mit gleichbleibender Geschwindigkeit auf Zielposition
- Trapez-Profil (Trapezoidal Profile) - Start mit ansteigender Geschwindigkeit bis zur Zielgeschwindigkeit für Zeitdauer t1, gleichbleibende Geschwindigkeit von t1 bis t2 sinkende Geschwindigkeit vor Erreichen des Ziels zum Zeitpunkt t3

Welches genutzt wird, ist abhängig von den Werten `Profile Velocity`und `Profile Acceleration`. Standard ist sofort auf Zielposition, was genutzt wird wenn `Profile Velocity` den Wert 0 hat.

| Profile Acceleration | Profile Velocity | Profile
| 0 | 0 | Step Instuction
| 0 | > 0 | Rectangular Profile
| > 0 | > 0 | Trapezoidal Profile

Die Profile können entweder auf Basis der Geschwindigkeit (Velocity-based) oder auf Basis der Zeit (Time-based) erzeugt werden. Dies wird durch ein Bit im `Drive Mode` festgelegt. Default ist Velocity-based.

Beim Rechteck-Profil:
- `Profile Acceleration` = 0
- `Profile Velocity` = Geschwindigkeit bei Velocity-based
                     = Zeitdauer für das Profil bei Time-based (t3)

Beim Trapez-Profile:
- `Profile Acceleration` = Beschleunigung bei Velocity-based
                       = Beschleunigungdauer bei Time-Based (t1)
- `Profile Velocity` = maximale Geschwindikgeit bei Velocity-based
                     = Zeitdauer für das Profil bei Time-based (t3)

Will man die Bewegung bis zum gewünchten Ziel mit einer bestimmten Geschwindigkeit (z.B. 10 Umdrehungen pro Minute) ausführen, nutzt man den `Drive Mode` `Velocity based` und setzt den Wert `Profile Velocity` auf 10 / 0,229 = 44.

Will man angeben, wie lange die Bewegung bis zum gewünschten Ziel dauern soll, wählt man als `Drive Mode` `Time-based` und gibt die Dauer der Bewegung in Millisekunden an.

Beim Trapezprofil gibt man beim Modus Velocity-based als `Profile Acceleration` die Beschleunigung an, dabei eintspricht 1 einer Beschleunigung von 214,58 Umdrehungen pro Minute². Als `Profile Velocity` gibt man die maximale Geschwindigkeit an.
Beispiel: `Profile Acceleration` =  4 (858,31 rev/min²) `Profile Velocity` = 100 (22,9 rev/min)

v = a * t
t = v/a = 0.02668 Min = 1,6 s


Die Time-based-Variante ist etwas anschaulicher: Hier gibt man die Beschleunigungsdauer und die Zeitdauer für das Profil insgesamt an.



## Die Methoden aus dem dynamixel_sdk

Die PortHandler-Klasse hat für das senden und Empfangen folgende Methoden:

readPort(self, length)
writePort(self, packet) - gibt die Länge in Byte zurück

Die writePort-Methode wird von folgender Methode der Klasse PacketHandler aufgerufen:

txPacket(self, port, txpacket)

In dieser Methode werden dem Datenpaket der Header und die Checksummen hinzugefügt und das Paket anschließend gesended



txPacket


rxPacket


Wichtige Return-Codes:
COMM_SUCCESS = 0



## Umsetzung in Python

Das Python-Dynamixel-SDK besteht aus einem Verzeichnis mit mehreren Modulen:

robotis_def.py
    enthält Konstanten zu Instruktionen und Kommunikationsergebnissen 
    und Methoden zu Umwandlung in Byte

port_handler.py
    enthält Methoden für das Versenden und Empfangen der Datenpakete


protocol2_packet_handler.py
    enthält Methoden zur Konstruktion der Datenpakete


### Methoden zur Konstruktion der Datenpakete

für Ping:
    - ping(port, id)
    - broadcastPing(port)

für Factory Reset
    - factoryReset(port, id, option)




# Videos / Links

https://www.youtube.com/watch?v=tkV9CPWjtgU

https://www.youtube.com/watch?v=YjARfu1ukv4

https://www.direcs.de/2020/02/ansteuerung-dynamixel-servo-mit-raspberry-pi-und-python-ohne-u2d2/