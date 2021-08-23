---
layout: post
title:  "Bluetooth am Raspberry Pi"
date:   2020-07-24 21:08:49 +0200
tags: fernsteuerung bluetooth raspberry python
---

Beim Raspberry Pi ist Bluetooth vorhanden und standardmäßig auch aktiviert. Man kann es wie folgt überprüfen:

```
$ sudo systemctl status bluetooth
```

## Bluetoothctl

Mit bluetoothctl kann Bluetooth über die Kommandozeile verwaltet werden:
```
$ bluetoothctl
```

Falls bluetoothctl nicht gefunden wird, kann man es über das Package bluez-utils installieren.

Innerhalb von bluetoothctl sieht dann der Prompt anders aus und man kann verschiedene Kommandos ausführen:

Bluetooth ggf. noch anschalten:
```
[bluetooth]# power on 
```

Das eigene Gerät sichtbar machen:
```
[bluetooth]# discoverable on 
```

Das eigene Gerät pairable machen:
```
[bluetooth]# pairable on 
```

Andere Geräte auflisten:
```
[bluetooth]# scan on 
```

Danach erscheinen die Bluetooth-Adressen:
```
...
[NEW] Device B8:27:EB:A6:96:88 ameise
...
```
Hier wird beispielsweise die Bluetooth-Schnittstelle eines anderen Raspis angezeigt. Um diesen zu pairen wird folgender Befehl eingegeben. Mit dem Pairen erteilt man den Geräten die Berechtigung miteinander zu kommunizieren:

```
[bluetooth]# pair B8:27:EB:A6:96:88
Attempting to pair with B8:27:EB:A6:96:88

[CHG] Device B8:27:EB:A6:96:88 ServicesResolved: yes
[CHG] Device B8:27:EB:A6:96:88 Paired: yes
Pairing successful
```

Informationen zu einem anderen Gerät kann man wie folgt anzeigen lassen:
```
[bluetooth]# info B8:27:EB:A6:96:88
Device B8:27:EB:A6:96:88 (public)
	Name: ameise
	Alias: ameise
	Class: 0x004c0000
	Paired: yes
	Trusted: no
	Blocked: no
	Connected: no
	LegacyPairing: no
	UUID: Headset                   (00001108-0000-1000-8000-00805f9b34fb)
	UUID: Audio Source              (0000110a-0000-1000-8000-00805f9b34fb)
	UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)
	UUID: A/V Remote Control Target (0000110c-0000-1000-8000-00805f9b34fb)
	UUID: A/V Remote Control        (0000110e-0000-1000-8000-00805f9b34fb)
	UUID: Headset AG                (00001112-0000-1000-8000-00805f9b34fb)
	UUID: Handsfree Audio Gateway   (0000111f-0000-1000-8000-00805f9b34fb)
	UUID: PnP Information           (00001200-0000-1000-8000-00805f9b34fb)
	Modalias: usb:v1D6Bp0246d0532
	RSSI: -66
	TxPower: 0
```

Weitere Erläuterungen siehe z.B. hier:
https://www.linux-magazine.com/Issues/2017/197/Command-Line-bluetoothctl 


## Serielle Verbindung über Bluetooth vom Android zum Raspi

https://www.reddit.com/r/raspberry_pi/comments/6nchaj/guide_how_to_establish_bluetooth_serial/

Für serielle Verbindungen über Bluetooth müssen wir dafür sorgen, dass der Bluetooth-Daemon im Kompabilitätsmodus gestartet wird und das auch der der Serial Port Service gestartet wird.

Hierzu editieren wir die Datei /etc/systemd/system/dbus-org.bluez.service und ergänzen den Eintrag 

```
ExecStart=/usr/lib/bluetooth/bluetoothd
```
wie folgt:
```
ExecStart=/usr/lib/bluetooth/bluetoothd -C
ExecStartPost=/usr/bin/sdptool add SP
```

Nach einem Reboot kann man dann prüfen ob der serielle Port über den Service-Discovery gefunden wird:
```
$ sudo sdptool browse local
```

Auf dem Raspi ist dann einzugeben:
```
[bluetooth]# discoverable on
Changing discoverable on succeeded
[CHG] Controller B8:27:EB:A6:96:88 Discoverable: yes
[bluetooth]# pairable on
Changing pairable on succeeded
[CHG] Device 58:CB:52:87:4F:AC Connected: yes
[CHG] Device 58:CB:52:87:4F:AC Connected: no
```
. 
Danach kann man auf dem Android-Phone unter Einstellungen-Verbundene Geräte den Raspi auswählen und "Koppeln" auswählen.

Wie man sieht, erscheint im Raspi das Android-Phone kurz als Connected: yes, dann aber wieder auf no. Das ist aber normal, denn wir müssen nun auf dem Raspi, die RFCOMM-Schnittstelle auf auf eingehende Verbindungen warten lassen:
```
$ sudo rfcomm watch hci0
Waiting for connection on channel 1
```

Eingehende serielle Verbindungen sind nun vom Android-Phone aus möglich. Es gibt dafür mehrere Apps. Hier im Beispiel verwenden wir die App "Serial Bluetooth Terminal":

* App installieren
* In der App unter "Devices" den Raspi auswählen.
* Im Terminal des Smartphones wird "Connected" angezeigt.

Die erfolgreiche Verbindung wird auch auf dem Raspi angezeigt:
```
Waiting for connection on channel 1
Connection from 58:CB:52:87:4F:AC to /dev/rfcomm0
Press CTRL-C for hangup
```

Texteingaben im "Serial Bluetooth Terminal" werden nun an den Raspi auf die serielle Schnittstelle /dev/rfcomm0 gesendet. Man kann sich diese dort mit folgendem Befehl anzeigen lassen:
```
$ cat /dev/rfcomm0 
```

Umgekehrt kann man vom Raspi Text an die serielle Schnittstelle übergeben:
```
$ echo 'gnampf' > /dev/rfcomm0 
```

## Serielle Verbindung über Bluetooth mit Python-Modul pyserial

Diese Variante setzt voraus, dass /dev/rfcomm0 existiert. Vor Start des Programms muss also - wie eben beschrieben - das Device (Smartphone) mit dem Raspi gepairt werden und auf dem Raspi
```
$ sudo rfcomm watch hci0
```
gestartet werden. Zudem ist die Verbindung in der App aufzubauen, so dass dort "Connected" erscheint.

Danach können einerseits Zeichen, die über die Verbindung empfangen werden, mit pyserial gelesen werden:

```python
import serial

ser = serial.Serial('/dev/rfcomm0')

while True:
    result = ser.read()
    print(result)
```

Andererseits können auch Daten über die Verbindung gesendet werden:

```python
import serial

ser = serial.Serial('/dev/rfcomm0')  
ser.write(b'hello') 
ser.close()   
```

## Bluetooth mit Python-Sockets

Die Python-Sockets-Implementierung unterstützt standardmäßig Bluetooth. Beispielcode für Server und Client ist hier zu finden: 
http://blog.kevindoran.co/bluetooth-programming-with-python-3/
https://people.csail.mit.edu/albert/bluez-intro/x232.html 

Unser folgendes Beispiel bluetooth-example-server.py nimmt vom Client einen String entgegen und sendet diesen doppelt zurück.

```python
import bluetooth

server_sock=bluetooth.BluetoothSocket( bluetooth.RFCOMM )

port = 1
server_sock.bind(("",port))
server_sock.listen(1)

size = 1024

try:
    client_sock, address = server_sock.accept()
    print("Accepted connection from ",address)
    while 1:
        data = client_sock.recv(size)
        if data:
            print("received [%s]" % data)
            client_sock.send(data+data)
except:	
    print("Closing socket")	
    client_sock.close()
    server_sock.close()
```

Man startet den Server auf dem Raspi mit:

$ python3 bluetoot-example-server.py 


Pairing vorausgesetzt, kann man anschließend mit der App "Serial Bluetooth Terminal" eine Verbindung aufbauen und Daten austauschen.

## Bluetooth mit Python-Modul PyBluez

Eine weitere Möglichkeit ist die Nutzung des Python-Moduls PyBluez. Auch hier gibt es Beispielcode für Server und Client: http://blog.kevindoran.co/bluetooth-programming-with-python-3/

Zuvor muss allerdings PyBluez installiert werden:

$ sudo apt-get install python3-bluez

Bei meinen Tests war eine Verbindung zwischen zwei Linux-Geräten (Raspi und Manjaro) mit PyBluez möglich. Was bei mir mit PyBluez nicht funktioniert hat, war den Raspi als Server zu betreiben und eine Verbindung vom Android-Gerät aufzubauen.

TODO: Fehlermeldung

## Bluetooth mit BlueDot

Sehr einfach funktioniert die Bluetooth-Kommunikation mittels BlueDot: https://github.com/martinohanlon/BlueDot 

BlueDot besteht aus
* der Android-App BlueDot
* dem Python-Modul BlueDot

Das Python-Modul installiert man auf dem Raspi über pip3:
```	
$ sudo pip3 install bluedot
```	

Vor der Nutzung muss man beide Geräte pairen.

Folgendes Python-Programm startet den Server, wartet auf verbindende Clients und wartet dann auf einen Knopfdruck. Wenn der blaue Knopf gedrückt wird, erscheint die Meldung "You pressed the blue dot!" im Terminal und das Programm wird beendet.

```python
from bluedot import BlueDot
bd = BlueDot()
bd.wait_for_press()
print("You pressed the blue dot!")
```	

Man kann aber nicht nur auf das Drücken des Buttons reagieren, sondern es gibt auch die Möglichkeit einer Nutzung des Buttons
* als Joystick (Hoch, Runter, Rechts und Links) zu verwenden
* als Slider
* für Wischbewegungen (Swiping)
* für Rotationsbewegungen

Die Doku enthält dafür entsprechenden Anwendungsbeispiele

Interessant ist, dass man die Bluetooth Comm API von BlueDot auch unabhängig von der BlueDot-App und damit mit anderen Apps nutzen kann. 

from bluedot.btcomm import BluetoothServer
from signal import pause

```	python
def data_received(data):
    print(data)
    s.send(data)

s = BluetoothServer(data_received)
pause()
```	


## Android-Apps

Es gibt einige Apps, mit denen man den Raspi oder Arduino über Bluetooth steuern kann. Hier eine Auswahl:

Serial Bluetooth Terminal:
* Vorwiegend für Textkommunikation (Terminal)
* Definition von Macro-Buttons möglich

Bluetooth RC Controller:
* Control-Pad für Fahrzeugsteuerung
* Zeichen für Steuerbefehle fest kodiert

Arduino Bluetooth:
* 4 Ansichten (LED-Controller, Terminal, Control-Pad, Buttons)
* Zeichen für Control-Pad und Buttons frei konfigurierbar
* enthält Werbung




## Weitere Links zum Thema Bluetooth mit Raspberry und Python


https://www.raspberrypi.org/forums/viewtopic.php?t=184711

https://lancaster-university.github.io/microbit-docs/ubit/radio/



WICHTIG  https://scribles.net/setting-up-bluetooth-serial-port-profile-on-raspberry-pi-using-d-bus-api/

https://stackoverflow.com/questions/41804222/cant-redirect-rfcomm-output-to-a-file

http://blog.kevindoran.co/bluetooth-programming-with-python-3/

https://roboticsknowledgebase.com/wiki/networking/bluetooth-sockets/

http://people.csail.mit.edu/rudolph/Teaching/Articles/PartOfBTBook.pdf 


Callipe Radio vs. Bluetooth



zu /dev/rfcomm
https://newbedev.com/how-to-i-connect-a-raw-serial-terminal-to-a-bluetooth-connection
http://www.koehlers.de/wiki/doku.php?id=pc:bluetooth


https://play.google.com/store/apps/details?id=com.crhostservices.com.crhostservices.androidbtcontrol&hl=gsw&gl=US 

https://hicraigchen.medium.com/access-raspberry-pi-terminal-via-bluetooth-on-android-device-9e51cc83221f


https://www.raspberry-pi-geek.com/Archive/2014/08/Getting-BLE-to-behave-on-the-Pi/(offset)/4


Hier ist eine gute Anleitung mit /dev/rfcomm
https://raspberrypi.stackexchange.com/questions/121216/how-to-best-handle-raspberry-pi-4-and-smartphone-connection-over-bluetooth-and-p


Client für Android erstellen mit App Inventor
https://www.youtube.com/watch?v=f4tnEBBlYAc

Eigene Adresse rausfinden:

$ hcitool dev

Adressen:
tuxli 68:54:5A:1A:AD:2D
libelle E4:5F:01:09:2F:5A



bluetooth]# info E4:5F:01:09:2F:5A
Device E4:5F:01:09:2F:5A (public)
	Name: libelle
	Alias: libelle
	Paired: yes
	Trusted: no
	Blocked: no
	Connected: no
	LegacyPairing: no
	UUID: A/V Remote Control Target (0000110c-0000-1000-8000-00805f9b34fb)
	UUID: A/V Remote Control        (0000110e-0000-1000-8000-00805f9b34fb)
	UUID: PnP Information           (00001200-0000-1000-8000-00805f9b34fb)
	Modalias: usb:v1D6Bp0246d0532


## Blueman


Mehr möglichkeiten hat man mit dem Bluetooth-Manager Blueman hat. Man installiert ihn wie folgt:

$ sudo apt-get install bluetooth  pi-bluetooth bluez blueman

Bei mir waren pi-pluetooth und bluez schon installiert.

Beim Start und Zugriff habe ich aber folgende Meldung erhalten:

org.freedesktop.DBus.Python.dbus.exceptions.DBusException: Not authorized

$ usermod -a pi -G netdev

und Anpassung der Datei /etc/polkit-1/rules.d/blueman.rules brachten keinen Erfolg.

