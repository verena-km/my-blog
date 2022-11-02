---
layout: post
title:  "E-Mails abfragen mit Python"
date:   2022-09-22
tags: Python IMAP E-Mail
---


Für das Abrufen von E-Mail wird zumeist das IMAP-Protokoll verwendet. Um das automatisiert mit einem Python-Skript zu machen kann man die Bibliothek imaplib nutzen. 

## Verbindung aufbauen

Bei unserem Mailserver läuft IMAP mit STARTTLS. Die Verbindung zum Server wird wie folgt aufgebaut:

```python
import imaplib
import ssl
# Die dem System bekannten Zertifikate laden
tls_context = ssl.create_default_context()

# Zum IMAP-Server
server = imaplib.IMAP4('meinmailserver.meinedomain.de')

# STARTTLS-Verschlüsselung starten
server.starttls(ssl_context=tls_context)

# Einloggen
server.login('testuser', 'geheimes_passwort')
```

Falls ein Zertifikat verwendet wird, dass dem System nicht bekannt ist, kann die Zertifikatsprüfung auschaltet werden.

```python
tls_context.check_hostname = False
tls_context.verify_mode = ssl.CERT_NONE
```
Nun kann man auf das Postfach zugreifen.

## Hinweis zur Nutzung der Bibliothek

Jedes Kommando der Bibliothek gibt ein Tuple zurück: (type, data). Dabei ist `type` normalerweise 'OK' oder 'NO' und `data` ist eine Liste, die die Antwort bzw. das Ergebnis des Kommandos enthält. Dabei ist `data` eine Liste, die oft aber nur ein Element enthält: entweder ein Byte-Objekt oder ein Tuple. 

Aus diesem Grund greifen wir nachfolgend oftmals auf das erste Element der Liste zu und wandeln die Byte-Objekte mit decode in Strings um

## Mailboxen auflisten

Die für den eingeloggten User verfügbaren Verzeichnisse kann man wie folgt auflisten lassen:

```python
tmp, mailboxes = server.list()
for mailbox in mailboxes:
    print(mailbox.decode("utf-8"))
```

## Mailbox auswählen

Um nach Inhalten zu suchen, muss zunächst eine Mailbox ausgewählt werden. Als Rückgabewert erhält man - wenn man will - die  Zahl der Nachrichten in dieser Mailbox. Setzt man `readonly` auf `True`, sind keine Änderungen erlaubt.

```python
tmp, count = server.select("INBOX", readonly=True)
print(count[0].decode("utf-8"))
```

## Nachrichten suchen

Sobald die Mailbox ausgewählt ist kann man nach Nachrichten suchen. Mit folgender Suche erhält man alle Nachrichten. Als Ergebnis erhält man einen ByteString mit den Nummern der Nachrichten durch Leerzeichen getrennt.

```python
tmp, data = server.search(None, 'ALL')
print(data[0])
```
Man kann auch speziellere Suchen durchführen, z.B. nach Nachrichten die einen bestimmten String im Betreff enthalten
```python
tmp, data = server.search(None,'(SUBJECT "Test")')
```
Weitere Suchmöglichkeiten ergeben sich aus dem [RFC3501](https://www.rfc-editor.org/rfc/rfc3501.html#section-6.4.4)

Möglich sind z.B. auch Suchen nach dem Datum und den folgenden Markierungen (FLAGS):

    \Seen: Nachricht gelesen
    \Answered: Nachricht beantwortet
    \Flagged: Nachricht als Wichtig markiert
    \Deleted: Nachricht als "zu löschen" markiert. Das tatsächliche löschen findet erst statt, wenn die expunge()-Funktion aufgerufen wird
    \Draft: Message has not completed composition (marked as a draft).

also z.B.
```python
tmp, data = server.search(None, 'SEEN')
tmp, data = server.search(None, 'UNSEEN')
tmp, data = server.search(None, 'ANSWERED')
tmp, data = server.search(None, 'UNANSWERED')
tmp, data = server.search(None, 'FLAGGED')
tmp, data = server.search(None, 'DELETED')
tmp, data = server.search(None, 'DRAFT')
```


## Nachrichten anzeigen lassen

Die Anzeige von Nachrichten basiert auf der vorherigen Auswahl einer Mailbox und einer Suchabfrage. Die Nachrichtennummern werden dann verwendet werden um die einzelnen Nachrichten abzurufen.

```python
tmp, data = server.search(None, 'ALL')
for num in data[0].split():
    tmp, data = server.fetch(num, '( BODY.PEEK[HEADER] FLAGS UID)')
    print(data)
```    

Beim Kommando fetch gibt man neben der Nachrichtennummer an, welche Teile der Nachricht zu holen sind. Die möglichen Werte findet man im [RFC3501](https://www.rfc-editor.org/rfc/rfc3501#section-6.4.5), z.B.

* BODY.PEEK[HEADER] für die Header-Felder
* BODY.PEEK[TEXT] für den Nachrichten-Text
* FLAGS für die gesetzten Flags
* UID für die eindeutige UID der Nachicht

Für die Weiterverarbeitung der E-Mail kann man aus den Rückgabewerten Message-Objekte erzeugen. Hierzu verwenden wir die Methode message_from_bytes aus der email-Bibliothek. Anschließend kann man einfach auf die einzelnen Bestandteile der E-Mail zugreifen

```python
import email
...
tmp, data = server.search(None, 'ALL')
for num in data[0].split():
    tmp, data = server.fetch(num, '( RFC822)')
    message = email.message_from_bytes(data[0][1])
    print("From       : {}".format(message.get("From")))
    print("To         : {}".format(message.get("To")))
    print("Bcc        : {}".format(message.get("Bcc")))
    print("Date       : {}".format(message.get("Date")))
    print("Subject    : {}".format(message.get("Subject")))
```    

## 

expunge

https://docs.python.org/3/library/imaplib.html
https://coderzcolumn.com/tutorials/python/imaplib-simple-guide-to-manage-mailboxes-using-python#11
https://www.atmail.com/blog/imap-commands/
http://pymotw.com/2/imaplib/
https://pymotw.com/3/imaplib/
https://techoverflow.net/2019/04/08/minimal-python-imap-over-tls-example/
https://deviloper.in/how-to-read-emails-using-python 