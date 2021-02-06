---
layout: post
title:  "Makecode-Projekte in Github verwalten"
date:   2021-02-04
tags: Makecode Calliope Github
---

[Github](https://github.com/) ist ein Dienst zur Versionsverwaltung unter Verwendung der Software [Git](https://de.wikipedia.org/wiki/Git). Durch die Nutzung einer Versionsverwaltung kann man mit mehreren Personen koordiniert an einem Projekt arbeiten und man hat die Möglichkeit, alle Änderungen nachzuvollziehen und ggf. wieder rückgängig zu machen.

Auch für Programme, die in Makecode erstellt werden, kann man Github zur Versionsverwaltung verwenden. 

## Vorgehensweise

### 1. Github-Account erstellen

Zuerst muss man einen (kostenlosen) [Github-Nutzer erstellen](https://github.com/join), sofern man nicht bereits einen hat.

### 2. Projekt in Makecode erstellen

In Makecode erstellt man wie gewohnt ein neues Projekt, indem man unter [https://makecode.calliope.cc/](https://makecode.calliope.cc/) auf "Neues Projekt" klickt, einen Namen für das Projekt angibt und dann auf "Erstelle" klickt.

### 3. Github-Repository erstellen

Aus Makecode heraus kann man dann ein Github-Repository erstellen, indem man auf den Button mit der Katze klickt.

![Github in Makecode](/images/screenshot_makecode_git.png) 

Nutzt man erstmalig Github in Makecode, wird dann aufgefordert, sich bei Github anzumelden. Man klickt auf "Anmelden" und gibt die Zugangsdaten ein, die man beim Erstellen des Github-Accounts gewählt hat.

Anschließend gibt man einen Repository-Namen und eine Repository-Beschreibung ein. Zudem kann man auswählen, ob das Repository öffentich, d.h. für alle zugänglich, oder privat sein soll.

Nach Eingabe dieser Daten klickt man auf "Los gehts" und das Repository wird bei Github errstellt. 

### 4. Änderungen durchführen und einchecken

Solange man keine Änderungen durchgeführt hat, wird neben der Katze ein Haken angezeigt. Nimmt man nun Änderungen vor, erscheint neben der Katze ein Pfeil.

Klickt man auf diesen Pfeil, bekommt man eine Übersicht der Änderungen angezeigt. Darin sind neu hinzugefügte, geänderte und gelöschte Teile zu erkennen. Man kann dann eine Beschreibung der Änderungen hinzufügen und diese durch Klick auf "Änderungen übernehmen und übertragen" auf Github speichern.

### 5. Änderungen anschauen und frühere Versionen wieder herstellen

Unter "Verlauf" kann man sich alle bisherigen nach Github übertragenen Änderungen (sog. Commits) anschauen und bei Bedarf auch frühere Versionen wiederherstellen.

### 6. Release veröffenlichen

Hat man einen bestimmten Entwicklungsstand erreicht, kann man diesem eine Versionskennzeichnung geben. Dafür klickt man im Bereich "Release-Zone" auf "Erzeuge Veröffentlichung" und wählt eine der drei Möglichkeiten für die Erzeugung der neuen Versionsnummer aus.

## Vorhandenes Repository nutzen

Hat man bereits ein Repository für ein Makecode-Projekt in Github erstellt und will an diesem weiterarbeiten, kann man auf der Eingangsseite von Makecode auf den Pfeil rechts oben "Ein Projekt importieren" klicken und "Dein GitHub Repository" auswählen.

Anschließend muss man auf "Anmelden" klicken, um sich auf Github anzumelden, danach landet man automatisch wieder in Makecode.

Man bekommt dann eine Liste der Makecode-Projekte des eigenen Nutzers in Github angezeigt und kann eines davon auswählen.


## Weitere Infos

Weitere Informationen und ein Video sind hier zu finden: [https://makecode.calliope.cc/github](https://makecode.calliope.cc/github)