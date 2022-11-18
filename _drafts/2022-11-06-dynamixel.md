---
layout: post
title:  "Dynamixel Motor "
date:   2022-11-06
tags: Motor Dynamixel
---

Dynamixel sind Servo-Motoren der Firma Robotis, die besonders für Robotik-Projekte geeignet sind.

Mehrere Dynamixel-Motoren können in Reihe geschaltet werden (sog. Daisy Chaining). Die Kommunikation läuft über die serielle Schnittstelle UART TTL.

Die Motoren kann man mit Hilfe des U2D2 Adapter mit einem PC verbinden.

Auch mit Microcontrollern kann man die Motoren ansteuern:
- mit speziellen Shields (z.B. Dynamixel Shield für Arduiono)
- über einen 
- direkt

Jeder der Motoren braucht eine eindeutige ID.

Spannungsversorgung

Ich verwende hier [Dynamixel vom Typ XL430-W250](https://emanual.robotis.com/docs/en/dxl/x/xl430-w250/)

Die Dynamixel-Motoren wird über das Senden und Empfangen von Datenpaketen über die serielle Schnittstelle programmiert. Das ist grundsätzlich mit allen Programmiersprachen möglich, das [Dynamixel SDK](https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_sdk/overview/) enthält Basisfunktionen und Beispiele für folgende Programmiersprachen:
* C, C++, C#
* Python
* Java
* MATLAB
* LAbView
* ROS




## Dynamixel-Wizard

Mit Hilfe des Dynamixel-Wizards kannn man sich mit dem D

Die Software für den Wizzard kann man hier herunterladen
https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_wizard2/

Unter Linux muss man die Datei zunächst noch ausführbar machen:

chmod u+x ./DynamixelWizard2Setup_x64

und kann den Wizzard dann starten.

Es erscheint ein Einrichtungsassistent, bei dem man Installationsordner auswählen muss. Nach der Einrichtung kann man den Wizzard starten.

((Bild))

Mit dem Scan kann man prüfen, welche Motoren vorhanden sind, sie werden dann links angezeigt.

In der Mitte sieht man dann die Register und deren Belegung.  Klickt man diese an erhält man rechts unten entsprechende Details und kann diese dort ggf. ändern.

Rechts oben wird eine Steuerung angzeigt. In diese kann man
* den Motor auf Werkseinstellungen zurücksetzen (factory Reset)
* einen Reboot durchführen
* den Betriebsmodus (Operating Mode) auswählen
* die Drehung (Torque) an- oder aus schalten
* die LED an- oder ausschalten

Abhängig vom Betriebsmodus wird ein Bedienelement angezeigt, in dem man die anzusteuernde Position oder die Geschwindigkeit wählen kann.

Die Veränderung einzelner Werte kann man sich in einem Graphen anzeigen lassen.

Packet???


https://www.daslhub.org/unlv/courses/me729/lesson-B-dynamixelWizard/lab/labXl320-introToDynamixelWizard-121419a.pdf

https://www.youtube.com/watch?v=JRRZW_l1V-U

https://www.youtube.com/watch?v=cQIDjrfDb24

## RoboPlusManager


## Kommunikation über Datenpakete

Der Dynamixel wird über das Senden und Empfangen von Datenpaketen über die serielle Schnittstelle programmiert. Für den XL430-W250 verwenden wir das [Protocol 2.0](https://emanual.robotis.com/docs/en/dxl/protocol2/).

Dort ist der Aufbau der Pakete beschrieben, unterschieden wird zwischen
* dem "Instruction Packet" das an das Gerät gesendet wird
* dem "Status Packet" das als Antwort zurück übermittelt wird.

Folgende "Instructions" können übermittelt werden:
* Ping
* Read
* Write
* RegWrite
* Action
* Factory Reset
* Reboot
* Clear
* Control Table Backup
* Status(Return)
* Sync Read
* Sync Write
* Fast Sync Read
* Bulk Read
* Bulk Write
* Fast Bulk Read


## Betriebsmodi


Es gibt vier Betriebsmodi, die durch den Wert in Register 11 bestimmt werden
* Velocity Control (Wert 1): Dauerhafte Drehbewegung
* Position (Wert 3, Default): Gezieltes Ansteuern von bestimmten Positionen
* Extended Position (Wert 4): Gezieltes Ansteuern von bestimmten Positionen mit mehrfachen Drehungen
* PWM: Dauerhafte Drehbewegung (?)





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