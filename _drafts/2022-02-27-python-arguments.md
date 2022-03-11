---
layout: post
title:  "Positions- und Schlüsselwortargumente in Python"
date:   2022-02-15
tags: Python
---

Parameter oder Argumente sind Variablen, die an eine Funktion oder Methode übergeben werden können. Von der Begrifflichkeit wird oft unterschieden zwischen Parametern, die in der Funktionsdefinition verwendet werden und Argumenten, die beim Aufruf der Funktion verwendet werden. Allerdings sind die Begrifflichkeiten nicht immer sauber abgegrenzt.

## Funktionsdefinition

Bei der Funktions- oder Methodendefinition in Python können die Parameter entweder ohne oder mit Standardwert angegeben werden.

In folgender Definition werden keine Standardwerte verwendet:

```python
def anrede1(vorname, nachname):
    print(f"Hallo {vorname} {nachname}")
```
In folgender Definition sind Standardwerte festgelegt:

```python
def anrede2(vorname="Max", nachname="Mustermann"):
    print(f"Hallo {vorname} {nachname}")
```

Es ist auch möglich beides gemischt zu verwenden, allerdings müssen in diesem Fall zunächst die Parameter ohne Standardwert aufgeführt werden
```python
def anrede3(vorname, nachname="Mustermann":
    print(f"Hallo {vorname} {nachname}")
```

## Funktionsaufrufe

Aufgerufen werden können Funktionen mit Parametern durch Übergabe der Werte an der jeweiligen Position oder aber durch Nutzung der Schlüsselwörter:

```python
anrede1("Harry", "Potter")
anrede1("Harry", nachname="Potter")
anrede1(nachname="Potter", vorname="Harry")
```

Wenn Standardwerte angegeben wurden, müssen diese Argumente nicht zwingend angegeben werden, erlaubt sind also auch folgende Aufrufe:
```python
anrede2()
anrede2("Harry")
anrede2(vorname="Harry")
anrede2("Harry")
anrede3("Harry")
```

Es sind also folgende Punkte zu beachten:
* Funktionen können mit Positions- oder mit Schlüsselwort-Argumenten aufgerufen werden.
* Alle Parameter ohne Standardwerte müssen angegeben werden.
* Werden die Argumente beim Aufruf ohne Schlüsselwort angegeben, müssen sie in der richtigen Reihenfolge sein.
* Beim Aufruf mit Schlüsselwort muss das Schlüsselwort als Parametername in der Funktionsdefinition vorhanden sein.

## Argumente beim Funktionsaufruf aus Liste oder Dictionary nehmen (Unpacking)

Die Argumente für den Funktionsaufruf kann man auch in eine Liste (oder ein Tupel) schreiben und diese dann mit dem `*`-Operator  als Einzelwerte "entpackt" an die Funktion übergeben:

```python
eingabe_list =  ["Harry","Potter"]
anrede1(*eingabe_list)
```
Die Liste muss mindestens so lang sein, wie die Anzahl der Parameter ohne Default-Wert, maximal aber so lang wie die Anzahl der Parameter insgesamt.

Eine weitere Möglichkeit ist, ein Dictionary mit den Parameternamen und -werten als Key-Value-Paare zu erstellen und die Werte dann mit dem `**`-Operator als Schlüsselwort-Argumente an die Funktion zu übergeben:
```python
eingabe_dict = {"vorname": "Harry","nachname": "Potter"}
anrede_pos(**eingabe_list)
```
Das Dictionary muss dabei mindestens alle Parameter ohne Default-Wert der Funktion mit Schlüssel und Wert enthalten. Schlüssel die in der Funktion nicht verwendet werden, dürfen nicht im Dictionary enthalten sein.

## Variable Anzahl von Parametern : Definition von Funktionen mit * und  ** (Packing)

Will man Funktionen mit einer variablen Anzahl von Parametern definieren, benötigt man ebenfalls den *- bzw den **-Operator. Meist werden dabei die Namen `*args` und `**kwargs` verwendet.

Bei `*args` wird eine Argumentliste variabler Länge ohne Schlüsselwörter an die Funktion übergeben:

```python
def anrede_mehrere(*args):
    print(type(args))
    for name in args:
        print(f"Hallo {name}")
```
```python
anrede_mehrere("Harry", "Hermine", "Ron")
```
Dabei ist `args` ein Tupel.

Bei `**kwargs` wird eine Argumentliste variabler Länge mit Schlüsselwörtern an die Funktion übergeben:

```python
def show(**kwargs):
    for key, value in kwargs.items():
        print(f"{key} - {value}")
```
```python
show(vorname="Harry", nachname="Potter", haus="Gryffindor")        
```
Dabei ist `kwargs` ein Dictionary.

## Keyword-only arguments

Wie wir oben gesehen haben, kann man bei Aufruf von Funktionen die Parameter nur über ihre Position (ohne Schlüsselwort) angeben. In Python3 gibt es aber auch die Möglichkeit, Funktionen so zu definieren, dass für entsprechende Parameter beim Aufruf zwingend das Schlüsselwort mit angegeben werden muss. Diese Parameter werden "Keyword-only arguments" genannt. 

Hierduch kann sichergestellt werden, dass der Nutzer einer Funktion nicht aus - aufgrund der falschen Position - einen falschen Parameter nutzt.

Dies kann dadurch erreicht werden, dass der *-Operator in der Funktionsdefinition genutzt wird. Alle diesem Packing-Argument nachfolgenden Argumente müssen beim Funktionsaufruf mit einem Schlüsselwort angegeben werden. Ohne Schlüsselwort würden diese nämlich dem Packing-Argument zugeschlagen werden.

```python
def anrede_mehrere2(*args, greeting):
    for name in args:
        print(f"{greeting} {name}")
```

```python
anrede_mehrere2("Harry", "Hermine", "Ron", greeting="Hiho")
```

Das ganze funktioniert aber auch ohne einen `*args`-Parameter. Wenn man diesen nicht braucht, kann man auch  einen einzelnen `*` in der Parameterliste der Funktionsdefinition nutzen. Alle auf diesen folgenden Argumente müssen mit Schlüsselwort aufgerufen werden.

```python
def anrede4(vorname, nachname, * , greeting):
    print(f"{greeting} {vorname} {nachname}!")
```

## Was nehmen?

Um sich 

* Macht es Sinn, dass der Nutzer der Funktion für einen bestimmten Parameter ein Schlüsselwort angeben kann oder gar muss?
* Gibt es für den Parameter einen sinnvollen Default-Wert?

Will man die Verwendung eines Schlüsselworts erzwingen, 


## Fehlermeldungen:


TypeError: anrede_pos() missing 1 required positional argument: 'vorname'


TypeError: anrede_pos() got multiple values for argument 'vorname'

* für einen Positionsparameter wurden


SyntaxError: keyword argument repeated


TypeError: anrede_key() takes from 0 to 2 positional arguments but 3 were given


TypeError: anrede_pos() got an unexpected keyword argument 'strasse'

TypeError: anrede_keyonly() missing 1 required keyword-only argument: 'greeting'


SyntaxError: non-default argument follows default argument


## Links
https://treyhunner.com/2018/10/asterisks-in-python-what-they-are-and-how-to-use-them/
https://python-3-for-scientists.readthedocs.io/en/latest/python3_advanced.html
https://einfachpython.de/python-kwargs-python-funktions-parameter/
https://stackabuse.com/unpacking-in-python-beyond-parallel-assignment/
https://medium.com/analytics-vidhya/the-magic-of-asterisks-in-python-aed3538deef9 
https://medium.com/analytics-vidhya/keyword-only-arguments-in-python-3c1c00051720