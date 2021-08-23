---
layout: post
title:  "Fernsteuerung über Bluetooth Classic am Raspberry Pi"
date:   2021-07-11 
tags: fernsteuerung bluetooth raspberry python
---

Der Raspberry Pi verfügt ab Version 3 über einen integrierten Bluetooth-Adapter, der sowohl Bluetooth Classic als auch Bluetooth Low Energy kann. Dieser Artikel beschäftigt sich mit Bluetooth Classic.

Ob Bluetooth läuft kann man wie folgt überprüfen:

```
$ sudo systemctl status bluetooth
```

Das Bluetooth-Interface kann man sich mit hciconfig anzeigen lassen, darin findet man auch die Bluetooth-Adresse:

```
$ hciconfig
```
bzw. noch ausführlicher mit

```
$ hciconfig -a
```

Verwendet man eine grapische Oberfläche kann man die Bluetooth-Verbindungen über diese verwalten. In der Taskbar gibt es dazu das Bluetooth-Symbol über das Bluetooth aktiviert und deaktiviert werden kann und über das man sich mit Bluettooth-Geräten verbinden kann.

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

## Fernsteuerung über Bluetooth

Der Raspi soll über Bluetooth ferngesteuert werden. Das steuernde Gerät (bespielsweise ein anderer Raspi oder ein Android-Gerät) sendet einen Befehl über Bluetooth und der Raspi erhält diesen und führt entsprechende Kommandos aus. Dabei soll möglichst ein Python-Programm verwendet werden. 

Im Internet finden sich hierfür unterschiedliche Herangehensweisen. Folgende habe ich auspropiert:

1. Erzeugung eines seriellen Ports für Bluetooth
2. Aufbauend auf Nr. 1 die Programmierung mit pyserial
Nutzung des Komplettpakets Bluedot
3. 


### 1. Erzeugung eines seriellen Ports für Bluetooth

Bei dieser Variante wird der Bluetooth-Daemon im Kompatibilitätmodus gestartet und auch der sog. Serial-Port-Service genutzt. Es wird dabei ein serieller Port emuliert über den Daten gesendet und empfangen werden können. Das Protokol nennt sich RFCOMM (Radio Frequency Communication).

Eine gute Anleitung ist in diesem Post zu finden:

https://raspberrypi.stackexchange.com/questions/121216/how-to-best-handle-raspberry-pi-4-and-smartphone-connection-over-bluetooth-and-p 

1. Bluetooth-Konfiguration anpassen

Hierzu editieren wir die Datei /etc/systemd/system/dbus-org.bluez.service und ergänzen den Eintrag 

```
ExecStart=/usr/lib/bluetooth/bluetoothd
```
wie folgt:
```
ExecStart=/usr/lib/bluetooth/bluetoothd -C
ExecStartPost=/usr/bin/sdptool add SP
```
Hierdurch wird Bluetooth im Kopatibilitätsmodus gestartet und der Service "Serial Port" dem Service Discovery Protocol hinzugefügt.

Danach laden wir die Konfiguration neu und starten den Bluetooth-Service neu:
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart bluetooth.service
```

Nun kann man prüfen, ob der serielle Port über den Service-Discovery gefunden wird.

```
$ sudo sdptool browse local
```
2. Geräte pairen

Auf dem Raspi ist dann einzugeben:

```
$ bluetoothctl

[bluetooth]# discoverable on
Changing discoverable on succeeded
[CHG] Controller E4:5F:01:09:2F:5A Discoverable: yes
[bluetooth]# pairable on
Changing pairable on succeeded
```

Danach kann man auf dem Android-Phone unter Einstellungen-Verbundene Geräte den Raspi auswählen und "Koppeln" auswählen.

Wie man sieht, erscheint im Raspi das Android-Phone kurz als Connected: yes, dann aber wieder auf no. Das liegt daran, dass der Raspi bislang noch nicht auf eingehende Verbindungen wartet.

3. Auf eingehende Verbindungen warten

Damit nun der Raspi auf eingehend Verbindungen wartet ist folgender Befehl einzugeben.

```
$ sudo rfcomm watch hci0
Waiting for connection on channel 1
```

4. Android-Phone verbinden

Eingehende serielle Verbindungen sind nun vom Android-Phone aus möglich. Hier kann man beispielsweise die App "Serial Bluetooth Terminal" nutzen

* App installieren
* In der App unter "Devices" den Raspi auswählen.
* Im Terminal des Smartphones wird "Connected" angezeigt.

Die erfolgreiche Verbindung wird auch auf dem Raspi angezeigt:
```
Waiting for connection on channel 1
Connection from 58:CB:52:87:4F:AC to /dev/rfcomm0
Press CTRL-C for hangup
```

5. Kommunikation 

Texteingaben im "Serial Bluetooth Terminal" werden nun an den Raspi auf die serielle Schnittstelle /dev/rfcomm0 gesendet.

Auf dem Raspi kann man beispielsweise mit picocom auf diese serielle Schnittstelle zugreifen und Daten darüber senden und empfangen
```
picocom /dev/rfcomm0
```
Picocom beendet man mit <Strg + a> + q.

6. Automatisierung

Damit der Raspi auch nach einem Reboot auf eingehende Bluetooth-Verbindungen wartet, kann man in der Datei /etc/rc.local folgendes eintragen (vor der letzten Zeile):

```
sudo bluetoothctl <<EOF
discoverable on
EOF

sudo rfcomm watch hci0 &
```

Zudem muss man noch in der Datei /etc/bluetooth/main.conf die Kommentarzeichen bei folgenden Zeilen entfernen um den Raspi dauerhaft über Bluetooth sichbar und pairbar zu machen:
```
discoverabletimeout=0 
pairabletimeout=0 
```


## Serielle Verbindung über Bluetooth mit Python-Modul pyserial

Diese Variante setzt voraus, dass /dev/rfcomm0 existiert. Vor Start des Programms muss also - wie eben beschrieben - das Device (Smartphone) mit dem Raspi gepairt werden und auf dem Raspi manuell oder automatisch
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



### Bluetooth mit BlueDot

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
