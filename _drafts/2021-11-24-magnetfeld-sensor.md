---
layout: post
title:  "Magnetfeldmesser / Kompassmodul"
date:   2021-11-24 
tags: Magnetfeldsensor I2C Raspberry
---

* TOC
{:toc}

## Magnetfeldsensoren

Magnetfeldsensoren ermitteln die Stärke des Magnetfelds in Richtung x-, y- und z-Achse. Einheiten für die magnetische Flussdichte sind Gauss oder Tesla, wobei 10.000 Gauß einem Tesla entsprechen. Vom Sensor erhält man drei Werte (für jede der drei Richtungen einen), die dann in Milligauß umgerechnet werden können. Dabei sind positive und negative Werte möglich.

![Magnetfeldsensor HMC5883L](/images/foto_magnetsensor.jpg)

Man kann Magnetfeldsensoren zur Erkennung von Magnetfeldern und als Kompass verwenden, was für die Orientierung von Robotern interessant macht. 

Aus den drei Meßwerten kann man die Kompassausrichtung wie folgt ermitteln: Die Werte in in x- und y-Achse kann man als kartesische Koordinaten betrachten und diese mit der arctan2-Funktion in Polarkoordinaten umrechnen. So erhält man den Winkel

```
φ = arctan2(x,y)
```

Aus den Werten x = -113 und y = 189 berechnet man beispielsweise einen Winkel von 120°. Bei den Werten x = 221 und y = 0 ist der Winkel 0°, d.h. der Sensor ist nach dem Erdmagnetfeld ausgerichtet, sofern keine Störeinflüsse vorhanden sind, die das Magnetfeld beieinflussen. 

Solche Einflüsse können entstehen
* durch Objekte, die selbst ein Magnetfeld erzeugen (Hard Iron Distortion)
* oder durch Objekte, die das Magnetfeld beieinflussen (Soft Iron Distortion). 

Betrachtet man die Messwerte der x- und y-Achse so erhält man beim Drehen des Sensors um die z-Achse idealerweise einen Kreis mit dem Mittelpunkt 0. Durch die oben genannte Einflüsse erhält man jedoch oft einen Kreis, dessen Mittelpunkt verschoben ist oder eine Ellipse.

Diese Einflüsse kann man durch Korrekturfaktoren beseitigen, die man durch eine Kalibrierung ermitteln kann. Näheres ist z.B. [hier](https://www.vectornav.com/resources/inertial-navigation-primer/specifications--and--error-budgets/specs-hsicalibration) zu finden.

Bei der Nutzung des Magnetfeldsensors als Kompass muss man zusätzlich die Deklination - also den Winkel zwischen den magnetischen Feldlinien und dem geographischen Nordpol berücksichtigen. Die ortabhängige Deklination kann man beispielsweise [hier](http://isdc.gfz-potsdam.de/igrf-declination-calculator/) abrufen, hier sind es aktuell 3° 4' bzw. dezimal 3,067.


## Magnetfeldsensor HMC5883L

Der oben abgebildete DEBO SENS MFELD GY-271 ist ein Modul für Magnetfeldmessung und Kompassfunktionen. Er kann über I2C an Raspberry, Arduino oder ESP32 angeschlossen werden.

Angeblich basiert er auf dem Sensor QMC5883L. Wenn man aber genauer hinschaut, hat der Chip die Beschriftung "L883", ist also nach folgenden Vergleichen ein HMC5883L von Honeywell.


* [https://belchip.by/sitedocs/10882.pdf](https://belchip.by/sitedocs/10882.pdf)
* [https://www.best-microcontroller-projects.com/hmc5883l.html](https://www.best-microcontroller-projects.com/hmc5883l.html)

Die beiden Chips sind ähnlich, aber nicht gleich, z.B. sind die Register z.T. unterschiedlich. Auch die Default-Adresse unterscheidet sich:
* beim HMC5883L - Default-Adresse 0x1e
* beim QMC5883L - Default-Adresse 0x0d

Damit kann man nicht nur an der Beschriftung sondern auch beim `i2detect` feststellen, welchen der beiden Chips man hat.

Das Datenblatt für den HMC5583L kann man beispielsweise [hier](https://cdn-shop.adafruit.com/datasheets/HMC5883L_3-Axis_Digital_Compass_IC.pdf) finden.

Der Sensor hat hat folgende Anschlusspins:
* VCC - 3,3 Spannung
* GND - Ground
* SCL - Taktleitung
* SDA - Datenleitung
* DRDY - Data Ready, Interrupt Pin

Folgende Register können gelesen bzw. geschrieben werden.

| Adress Location | Name | Access |
|-----------------|------|--------|
|00|Configuration Register A|  Read/Write|
|01|Configuration Register B|  Read/Write| 
|02|Mode Register| Read/Write |
|03|Data Output X MSB Register| Read |
|04|Data Output X LSB Register| Read |
|05|Data Output Z MSB Register| Read |
|06|Data Output Z LSB Register| Read |
|07|Data Output Y MSB Register| Read |
|08|Data Output Y LSB Register| Read |
|09|Status Register| Read |
|10|Identification Register A| Read |
|11|Identification Register B| Read |
|12|Identification Register C| Read |
 
Über das Configuration Register A kann unter anderem konfiguriert werden, wie häufig neue Daten in die Output-Register geschrieben werden und wie Mittelwerte über die Messungen gebildet werden. 

Mit dem Configuration Register B können die Werte skaliert werden.

Das Mode-Register legt den Mess-Modus des Geräts fest. Es gibt einen Contiuous-Measurement-Modus, bei dem dauerhaft gemessen und die Ergebnisse in die Register geschrieben werden. Beim Single-Measurement-Modus wird immer nur eine Messung durchgeführt.

Das Status-Register enthält Statusanzeigen für das Gerät.


##  Magnetfeldsensor HMC5883L am Raspberry Pi

Wir schließen den Sensor wie folgt an den Raspberry Pi an:

* VCC an Pin 1 (3.3.V)
* SDA an Pin 3 
* SCL an Pin 5 
* GND an pin 7 

Den DRDY-Pin des Sensors nutzen wir nicht.

Falls noch nicht aktiv, muss I2C mit `raspi-config` unter `Interface Options - P5 I2C` noch aktiviert werden.

Mit 
```
$ i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- 1e -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --    
```
sieht man, dass der Sensor an Adresse `1e` angeschlossen ist

### Beispiele und Bibliotheken

Für den Rasperry Pi sind im Intenet mehrere Beispiele und Bibliotheken zu finden. Einige basieren auf `smbus`, andere auf anderen Biblotheken. Unterschiede liegen zudem darin, welche Funktionen des Sensors (z.B. Skalierung) umgesetzt sind und ob Werte wie z.B. die Kompassausrichtung berechnet werden.

Hier einige Bibliotheken die auf `smbus` basieren.

* [https://github.com/Taur-Tech/HMC5883L/tree/master/libhmc5883l](https://github.com/Taur-Tech/HMC5883L/tree/master/libhmc5883l)
    * berücksichtigt unterschiedliche Skalierung
    * keine Kompasswerte
* [https://github.com/rm-hull/hmc5883l](https://github.com/Taur-Tech/HMC5883L/tree/master/libhmc5883l)
    * berücksichtig unterschiedliche Skalierung
    * Kompasswerte
* [https://github.com/ControlEverythingCommunity/HMC5883](https://github.com/ControlEverythingCommunity/HMC5883)
    * keine Skalierung
    * keine Kompasswerte
* [https://www.electronicwings.com/raspberry-pi/triple-axis-magnetometer-hmc5883l-interfacing-with-raspberry-pi](https://www.electronicwings.com/raspberry-pi/triple-axis-magnetometer-hmc5883l-interfacing-with-raspberry-pi)
    * keine Skalierung
    * Kompasswerte

Folgendes Projekt enthält auch Code zum Kalibrieren des Kompasses, allerdings ist es für den QMC5883L: [https://github.com/RigacciOrg/py-qmc5883l](https://github.com/RigacciOrg/py-qmc5883l)


### Eigene Umsetzung

Auf Basis der oben genannten Beispiele und Projekte habe ich eine eigene Klase für den HMC588L erstellt.

```python
# -*- coding: utf-8 -*-
"""
Python driver for the HMC5883L 3-Axis Magnetic Sensor.
(based on https://github.com/RigacciOrg/py-qmc5883l)
"""

import smbus
from time import sleep
import math
import logging


DFLT_BUS = 1
DFLT_ADDRESS = 0x1e

#some MPU6050 Registers and their Address
REG_A     = 0              #Address of Configuration register A
REG_B     = 0x01           #Address of configuration register B
REG_MODE  = 0x02           #Address of mode register

REG_XOUT_LSB = 0x04      # Output Data Registers for magnetic sensor.
REG_XOUT_MSB = 0x03
REG_YOUT_LSB = 0x08
REG_YOUT_MSB = 0x07
REG_ZOUT_LSB = 0x06
REG_ZOUT_MSB = 0x05

REG_STATUS = 0x09 # not used yet

# Flags for Register A
# Data Output Rate (Hz)
ODR_0 = 0b00000000   # 0.75 Hz
ODR_1 = 0b00000100   # 1.5 Hz
ODR_2 = 0b00001000   # 3 Hz
ODR_3 = 0b00001100   # 7.5 Hz
ODR_4 = 0b00010000   # 15 Hz
ODR_5 = 0b00010100   # 30 Hz
ODR_6 = 0b00011000   # 75 Hz

# number of samples averaged
AVERAGE_1 = 0b00000000 # 1
AVERAGE_2 = 0b00100000 # 2
AVERAGE_4 = 0b01000000 # 4
AVERAGE_8 = 0b01100000 # 8

# Flags for Register B
# Range [register_value, digital_resolution]

RANGE_0 = [0b00000000,  .73]
RANGE_1 = [0b00100000,  .92]
RANGE_2 = [0b01000000,  1.22]
RANGE_3 = [0b01100000,  1.52]
RANGE_4 = [0b10000000,  2.27]
RANGE_5 = [0b10100000,  2.56]
RANGE_6 = [0b11000000,  3.03]
RANGE_7 = [0b11100000,  4.03]

# Flags for Register Mode
MODE_CONT = 0b00000000 # Continuous-Measurement Mode
MODE_SING = 0b00000001 # Single-Measurement Mode - not used yet
MODE_IDLE = 0b00000010 # Idle Mode - not used yet

class HMC588L(object):
    """ Constructor for sensor object"""
    def __init__(self, 
                i2c_bus=DFLT_BUS,
                address=DFLT_ADDRESS,
                output_data_rate = ODR_5,
                samples_averaged = AVERAGE_1,
                range = RANGE_7):
        self.address = address
        self.bus = smbus.SMBus(i2c_bus)
        self.range = range

        self._declination = 0.0
        self._calibration = [[1.0, 0.0, 0.0],
                        [0.0, 1.0, 0.0],
                        [0.0, 0.0, 1.0]]

        #write to Configuration Register A
        reg_a_value = (output_data_rate | samples_averaged)
        self.bus.write_byte_data(DFLT_ADDRESS, REG_A, reg_a_value)

        #Write to Configuration Register B for gain
        self.bus.write_byte_data(DFLT_ADDRESS, REG_B, self.range[0])

        #Write to mode Register for selecting mode
        self.bus.write_byte_data(DFLT_ADDRESS, REG_MODE, MODE_CONT)

        # wait 67ms - see example in datasheet
        sleep(0.067)


    def read_raw_data(self,addr):
        """Get raw data from addr an and addr+1 """
        # höherwertige 8 bit an der ersten Adresse, niedrige an der Folgeadresse
        high = self.bus.read_byte_data(DFLT_ADDRESS, addr)
        low = self.bus.read_byte_data(DFLT_ADDRESS, addr+1)

        # concatenate higher and lower value
        # höherwertige 8 Bits um 8 Stellen nach nach links schieben
        # und dann mit den niedrigen ODER-verknüpfen
        value = ((high << 8) | low)

        # to get signed value from module
        # value ist ein in Dez. umgerechneter Zweierkomplement-Wert
        if(value > 32768): # also Zahl negativ (binär steht vorne eine 1)
            value = value - 65536
        return value

    def get_magnet_raw(self):
        """Get the 3 axis raw values from magnetic sensor.""" 
        x = self.read_raw_data(REG_XOUT_MSB)
        z = self.read_raw_data(REG_ZOUT_MSB)
        y = self.read_raw_data(REG_YOUT_MSB)
        return [x,y,z]

    def get_magnet_mG(self):
        """Get the 3 axis values from magnetic sensor converted to milliGauss."""      
        [x,y,z] = self.get_magnet_raw()
        return [x*self.range[1],y*self.range[1],z*self.range[1]]

    def get_magnet(self):
        """Return the horizontal magnetic sensor vector with (x, y) calibration applied."""
        [x, y, z] = self.get_magnet_mG()
        if x is None or y is None:
            [x1, y1] = [x, y]
        else:
            c = self._calibration
            x1 = x * c[0][0] + y * c[0][1] + c[0][2]
            y1 = x * c[1][0] + y * c[1][1] + c[1][2]
        return [x1, y1]        

    def get_bearing_raw(self):
        """Horizontal bearing (in degrees) from magnetic value X and Y."""
        [x, y, z] = self.get_magnet_raw()
        if x is None or y is None:
            return None
        else:
            b = math.degrees(math.atan2(y, x))
            if b < 0:
                b += 360.0
            return b

    def get_bearing(self):
        """Horizontal bearing, adjusted by calibration and declination."""
        [x, y] = self.get_magnet()
        if x is None or y is None:
            return None
        else:
            b = math.degrees(math.atan2(y, x))
            if b < 0:
                b += 360.0
            b += self._declination
            if b < 0.0:
                b += 360.0
            elif b >= 360.0:
                b -= 360.0
        return b

    def set_declination(self, value):
        """Set the magnetic declination, in degrees."""
        try:
            d = float(value)
            if d < -180.0 or d > 180.0:
                logging.error(u'Declination must be >= -180 and <= 180.')
            else:
                self._declination = d
        except:
            logging.error(u'Declination must be a float value.')

    def get_declination(self):
        """Return the current set value of magnetic declination."""
        return self._declination        


    def set_calibration(self, value):
        """Set the 3x3 matrix for horizontal (x, y) magnetic vector calibration."""
        c = [[1.0, 0.0, 0.0], [0.0, 1.0, 0.0], [0.0, 0.0, 1.0]]
        try:
            for i in range(0, 3):
                for j in range(0, 3):
                    c[i][j] = float(value[i][j])
            self._calibration = c
        except:
            logging.error(u'Calibration must be a 3x3 float matrix.')

    def get_calibration(self):
        """Return the current set value of the calibration matrix."""
        return self._calibration

    declination = property(fget=get_declination,
                           fset=set_declination,
                           doc=u'Magnetic declination to adjust bearing.')

    calibration = property(fget=get_calibration,
                           fset=set_calibration,
                           doc=u'Transformation matrix to adjust (x, y) magnetic vector.')

```
Über die Parametern des Konstrukturs kann man die Messhäufigkeit (data output rate), die Durchschnittsbildung und den Wertebereich festlegen. Mit der Methode `get_magnet_raw` erhält man die drei Werte aus den Registern des Sensors als Liste. Mit `get_magnet_mG` die in Milli-Gauss umgerechneten Werte.

Mit `set_calibration` kann man eine 3x3-Matrix für eine Kalibrierung bezüglich der x- und y- Achse festlegen. Mit `get_magnet` erhält man dann die entspreched der Kalibrierung korrigierten Werte. Wie man die Matrix bestimmt, wird später erläutert.

Mit `set_declination` kann man den Deklinations-Wert setzen. Um die Kompassausrichtung zu erhalten gibt es die Methode `get_bearing_raw`, welche die Kompassausrichtung basierend auf den Rohdaten des Sensors berechnet. Daneben gibt es noch die Methode `get_bearing`, über die man die Kompassausrichtung auf Basis der kalibrierten Werte und mit Berücksichtigung der Deklination erhält.


Die Klasse für den Sensor kann man dann wie folgt nutzen und sich zunächst die unkalibrierten Werte ausgebeben lassen.

```python
if __name__ == "__main__":
    sensor = HMC588L()

    while True:
        sensor_values = sensor.get_magnet_raw()
        print(f"Magnetfeldwerte in mG: x={sensor_values[0]:.2f} y={sensor_values[1]:.2f} z={sensor_values[2]:.2f}")
        print(f"Kompassausrichtung           : {sensor.get_bearing_raw():.0f}")
        sleep(1)
```


### Kalibrierung

Zum Kalibrieren habe ich die Skripte von [RigacciOrg / py-qmc5883l](https://github.com/RigacciOrg/py-qmc5883l/tree/master/calibration) verwendet. Dabei muss man im Skript `2d-calibration-get-samples` als Sensor ein Objekt der Klasse `HMC588L` statt eines der Klasse `QMC5883L` erzeugen, ansonsten kann man gemäß der Anleitung vorgehen:

* Mit dem angepassten ersten Skript die Beispieldaten durch Drehen des Sensors erzeugen. Diese werden dann in eine Datei geschrieben.
* Mit dem zweiten Skript `2d-calibration-make-calc` aus den Beispieldaten die Matrix mit den Korrekturfaktoren berechnen. Das Skript erzeugt ein Skript für das Tool `gnuplot` mit dem Messdaten und mathematische Funktionen visualisiert werden können.
* Anschließend kann man das Gnuplot-Skript aufrufen. Es erzeugt eine png-Datei mit der Visualisierung.
* Die Korrekturmatrix kann man zur weiteren Verwendung ebenfalls aus dem Gnuplot-Skript ablesen.

Eventuell muss gnuplot noch installiert werden.
```
apt-get install gnuplot
```

Bei meinem Sensor hat sich folgendes Bild ergeben:
![Gnuplot Magnetsensor](/images/magnet-data_20211210_1311.png)

Die berechnete Transformationsmatrix hat folgende Werte:
```
[[ 1.00133880e+00 -3.08894869e-03  1.23058858e+01]
 [-3.08894869e-03  1.00712699e+00  2.12515715e+01]
 [ 0.00000000e+00  0.00000000e+00  1.00000000e+00]]
```

Die Matrix kann man dann wie folgt für die Berechung der Werte verwenden:

```python
if __name__ == "__main__":
    sensor = HMC588L()
    sensor.calibration = [[1.0054814113626533, -0.002815058983808072, 277.1905648362572], [-0.002815058983808072, 1.0014457147179852, 139.0017853354902], [0.0, 0.0, 1.0]]

    while True:

        sensor_values = sensor.get_magnet()
        print(f"Magnetfeldwerte in mG: x={sensor_values[0]:.2f} y={sensor_values[1]:.2f} z={sensor_values[2]:.2f}")
        print(f"Kompassausrichtung: {sensor.get_bearing_raw():.0f}")
        sleep(1)
```

### Alternative Kalibrierung

Bei der Nutzung des Sensors als Kompass zeigten sich trotz der Kalibrierung noch Probleme: Richtet man den Kompass jeweils in bestimmten Winkeln z.B. nach den Himmelsrichtungen 0°, 90°, 180° und 270° aus, so weichen die vom Sensor gemessenen Winkel zum Teil erheblich davon ab. 

Für die Verwendung des Kompasses zur Orientierung auf einem mobilen Fahrzeug nutzen wir daher eine andere Methode zur Kalibrierung:
* Wir ermitteln den gemessenen Winkel in bestimmten tatsächlichen Winkeln, z.B. 0°, 90°, 180° und 270°.
* Dann berechnen wir die jeweiligen Differenzen der gemessenen Winkel und erhalten dann für jedes "Segment" einen Wert dafür, um wieviel Grad sich der gemessene Wert beim tatsächlichen Positionswechsel um ein Grad verändert
* Diese Korrekturwerte pro Segment können wir dann verwenden, um aus den gemessenen Werten die tatsächlichen Werte zu berechnen.

Zum Einlesen der Werte definieren wir eine Methode `calibrate_positions`, die den Nutzer bei Aufruf interaktiv dazu auffordert zunächst die Zahl der Messwerte einzugeben. Danach gibt die Methode jeweils den Winkel vor, auf den der Sensor zu platzieren ist und fordert den Nutzer auf, die Positionierung zu bestätigen. Durch Aufruf von `get_bearing_raw` werden die Messwerte bestimmt und in eine Liste eingelesen.

Aus dieser Liste wird in der Methode `get_conversion_list` dann eine weitere Liste generiert, die zu jedem Segment den gemessenen Anfangs- und Endwert sowie den Faktor enthält, mit dem man auf den tatsächlichen Winkel umrechnen kann. Diese Liste wird in einer json-Datei abgespeichert.

Bei der Erzeugung des Sensor-Objekts wird geprüft, ob schon eine Datei mit Kalibrierung vorliegt und die darin enthaltene Liste als Instanzvariable geladen.

Ermittelt man nun den Winkel mit der Methode `get_bearing`, wird geprüft, ob bereits eine Kalibrierung durchgeführt wurde. Falls nein, wird eine entsprechende Meldung ausgegeben und der gemessene Wert unverändert ausgegeben. Falls eine Kalibrierung vorhanden ist, wird mit Hilfe der Umrechnungsliste aus dem gemessenen Wert der tatsächliche Wert berechnet und zurückgegeben.

Hier der Programmcode mit alternativer Kalibrierung:
```python
# -*- coding: utf-8 -*-
"""
Python driver for the HMC5883L 3-Axis Magnetic Sensor.
(based on https://github.com/RigacciOrg/py-qmc5883l)
with special calibration
"""

import smbus
from time import sleep
import math
import logging
import json


DFLT_BUS = 1
DFLT_ADDRESS = 0x1e

#some MPU6050 Registers and their Address
REG_A     = 0              #Address of Configuration register A
REG_B     = 0x01           #Address of configuration register B
REG_MODE  = 0x02           #Address of mode register

REG_XOUT_LSB = 0x04      # Output Data Registers for magnetic sensor.
REG_XOUT_MSB = 0x03
REG_YOUT_LSB = 0x08
REG_YOUT_MSB = 0x07
REG_ZOUT_LSB = 0x06
REG_ZOUT_MSB = 0x05

REG_STATUS = 0x09 # not used yet

# Flags for Register A
# Data Output Rate (Hz)
ODR_0 = 0b00000000   # 0.75 Hz
ODR_1 = 0b00000100   # 1.5 Hz
ODR_2 = 0b00001000   # 3 Hz
ODR_3 = 0b00001100   # 7.5 Hz
ODR_4 = 0b00010000   # 15 Hz
ODR_5 = 0b00010100   # 30 Hz
ODR_6 = 0b00011000   # 75 Hz

# number of samples averaged
AVERAGE_1 = 0b00000000 # 1
AVERAGE_2 = 0b00100000 # 2
AVERAGE_4 = 0b01000000 # 4
AVERAGE_8 = 0b01100000 # 8

# Flags for Register B
# Range [register_value, digital_resolution]

RANGE_0 = [0b00000000,  .73]
RANGE_1 = [0b00100000,  .92]
RANGE_2 = [0b01000000,  1.22]
RANGE_3 = [0b01100000,  1.52]
RANGE_4 = [0b10000000,  2.27]
RANGE_5 = [0b10100000,  2.56]
RANGE_6 = [0b11000000,  3.03]
RANGE_7 = [0b11100000,  4.03]

# Flags for Register Mode
MODE_CONT = 0b00000000 # Continuous-Measurement Mode
MODE_SING = 0b00000001 # Single-Measurement Mode - not used yet
MODE_IDLE = 0b00000010 # Idle Mode - not used yet

class HMC588L(object):
    """ Constructor for sensor object"""
    def __init__(self, 
                i2c_bus=DFLT_BUS,
                address=DFLT_ADDRESS,
                output_data_rate = ODR_6,
                samples_averaged = AVERAGE_1,
                range = RANGE_7):
        self.address = address
        self.bus = smbus.SMBus(i2c_bus)
        self.range = range
        self.seg_list = None

        #write to Configuration Register A
        reg_a_value = (output_data_rate | samples_averaged)
        self.bus.write_byte_data(DFLT_ADDRESS, REG_A, reg_a_value)

        #Write to Configuration Register B for gain
        self.bus.write_byte_data(DFLT_ADDRESS, REG_B, self.range[0])

        #Write to mode Register for selecting mode
        self.bus.write_byte_data(DFLT_ADDRESS, REG_MODE, MODE_CONT)

        # wait 67ms - see example in datasheet
        sleep(0.067)

        # read calibration list from file
        try:
            f = open('data.json')
            self.seg_list = json.loads(f.read())
            print(self.seg_list)
            f.close()
        except FileNotFoundError:
            print('Keine Kalibrierungsdaten gefunden.')

    def read_raw_data(self,addr):
        """Get raw data from addr an and addr+1 """
        # höherwertige 8 bit an der ersten Adresse, niedrige an der Folgeadresse
        high = self.bus.read_byte_data(DFLT_ADDRESS, addr)
        low = self.bus.read_byte_data(DFLT_ADDRESS, addr+1)

        # concatenate higher and lower value
        # höherwertige 8 Bits um 8 Stellen nach nach links schieben
        # und dann mit den niedrigen ODER-verknüpfen
        value = ((high << 8) | low)

        # to get signed value from module
        # value ist ein in Dez. umgerechneter Zweierkomplement-Wert
        if(value > 32768): # also Zahl negativ (binär steht vorne eine 1)
            value = value - 65536
        return value

    def get_magnet_raw(self):
        """Get the 3 axis raw values from magnetic sensor.""" 
        x = self.read_raw_data(REG_XOUT_MSB)
        z = self.read_raw_data(REG_ZOUT_MSB)
        y = self.read_raw_data(REG_YOUT_MSB)
        return [x,y,z]

    def get_magnet_mG(self):
        """Get the 3 axis values from magnetic sensor converted to milliGauss."""      
        [x,y,z] = self.get_magnet_raw()
        return [x*self.range[1],y*self.range[1],z*self.range[1]]


    def get_bearing_raw(self):
        """Horizontal bearing (in degrees) from magnetic value X and Y."""
        [x, y, z] = self.get_magnet_raw()
        if x is None or y is None:
            return None
        else:
            b = math.degrees(math.atan2(y, x))
            if b < 0:
                b += 360.0
            return b
   
    def get_bearing(self):
        """Calibrated Horizontal bearing (in degrees) from magnetic value X and Y """
        value = self.get_bearing_raw()

        if not self.seg_list:
            print("Sensor is not calibrated.")
            return value

        else:
            # Wert ggf. um 360 Grad erhoehen
            if value < self.seg_list[0]['start_value']:
                value = value + 360

            # Wert umrechnen und zurueckgeben
            for segment in self.seg_list:
                if value >= segment['start_value'] and value <= segment['end_value']:
                    return (value - segment['start_value']) / segment['divisor'] + segment["target_value"]

    def calibrate_positions(self):
        """Start interactive callibration dialog and save data in file """       
        position_count = int(input("Anzahl der Positinen: "))
        seg_size = 360 / position_count
        mod_list=[]
        for position in range(position_count):
            input(f"Stelle den Sensor auf {position*seg_size} Grad und bestätige mit Enter")
            mod_list.append(self.get_bearing_raw())

        self.seg_list = self.get_conversion_list(mod_list)
        self.safe_calibration()


    def get_conversion_list(self,list):
        """Create conversion list from list of angles"""
        seg_size = 360 / len(list)
        # Liste um Zielwerte ergänzt 
        seg_list = []
        target_value = 0
        for element in list: 
            segment = {"start_value":element,"target_value": target_value}
            target_value = target_value + seg_size
            seg_list.append(segment)

        # sortieren
        sorted_seg_list = sorted(seg_list, key=lambda d: d['start_value']) 

        # Liste um Endwerte und Divisor ergänzen
        for idx, element in enumerate(sorted_seg_list):
            start = element["start_value"]
            if idx < len(sorted_seg_list)-1:
                end = sorted_seg_list[idx+1]["start_value"]
            else:
                end = sorted_seg_list[0]["start_value"]+360
            element["end_value"] = end
            divisor = (end - start) / seg_size
            element["divisor"] = divisor

        print(sorted_seg_list)
        return(sorted_seg_list)

    def safe_calibration(self):
        """Save conversion list to file"""
        with open('data.json', 'w', encoding='utf-8') as f:
            json.dump(self.seg_list, f, ensure_ascii=False, indent=4)

    def has_calibration(self):
        """Check a calibration is saved"""
        return self.seg_list is not None


if __name__ == "__main__":
    sensor = HMC588L()
    if not sensor.has_calibration():
        sensor.calibrate_positions()

    while True:
        sensor_values = sensor.get_magnet_mG()
        print(f"Magnetfeldwerte in mG:         x={sensor_values[0]:.2f} y={sensor_values[1]:.2f}, z={sensor_values[2]:.2f}")
        print(f"Kompassausrichtung:            {sensor.get_bearing_raw():.0f}")
        print(f"Kompassausrichtung korrigiert: {sensor.get_bearing():.0f}")
        print()

        sleep(1)
```
