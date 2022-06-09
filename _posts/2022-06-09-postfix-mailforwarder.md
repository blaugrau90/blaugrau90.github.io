---
title: Postfix Mailforwarder unter Linux einrichten
date: 2022-06-09 17:30:00 +200
categories: [Homelab, Linux]
tags: [postfix,mailforwarder,mail,linux,smtp]
---

# Postfix Mailforwarder

In diesem Artikel wird Postfix so konfiguriert, dass es E-Mails über einen externen SMTP-Server weiterleitet. Der Vorteil daran ist, dass man seinen Mailserver nicht selbst auf dem System hosten muss, sondern Mails über einen externen Mailserver (z.B. [web.de](http://web.de/), [gmx.de](http://gmx.de/), etc.) versendet.

Da das Versenden von E-Mails über Postifx aufgrund der Spam-Filter und der von den E-Mail-Anbietern in den letzten Jahren eingeführten Beschränkungen immer schwieriger geworden ist, wird empfohlen alle Postfix/PHP mail()-E-Mails über einen externen, vertrauenswürdigen E-Mail-Anbieter weiterzuleiten, um die Zustellung zu gewährleisten.


## Vorbereitung

Es wird die Adresse und der Port des SMTP-Servers sowie der Benutzernamen und das Kennwort für das E-Mail-Konto benötigt. \n Der SMTP-Port sollte 587 sein, kann aber je nach Ihrem Host unterschiedlich sein.


## Installation

Zu Installation von Postfix werden zunächst einige Pakete benötigt, die sich mit den untenstehenden Befehlen installieren lassen. in den `mailutils` befindet sich unter anderem auch das *Postfix* Paket, um das es hier geht.

```bash
sudo apt-get update
sudo apt install -y mailutils
```

Während der Installation von *Postfix* wird auch die Grundkonfiguration vorgenommen. Diese sollte bei einem externen Mailserver, wie in diesem Beispiel, wie auf dem Bild zu sehen, eingestellt werden.

 ![Postfix Configuration](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-06-09-postfix/postfix01.png?raw=true)
 _Postfix Configuration_


 ![Internet Site](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-06-09-postfix/postfix02.png?raw=true)
_Internet Site auswählen_


Hier wählt man **Internet Site** aus, da die Mails ja zu einem Mailserver im Internet transferiert werden.

 ![Eingabe des Mailservers](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-06-09-postfix/postfix03.png?raw=true)
 _Eingabe des Mailservers_
 

Nach der Eingabe der FQDN, ist die Grundinstallation abgeschlossen.


## Konfiguration von Postfix

Zunächst muss in der *Postfix* `main.cf` Datei eine Anpssung gemacht werden.In der Datei stehen zu Beginn schon einige Konfigurationen. Scrollt man ganz nach unten gibt es hier den Eintrag `relayhost=` - dieser muss entfernt werden.Anschließend fügt man ganz am Ende der Datei folgenden Codeschnipsel ein. Dabei muss bei `relayhost=` zwischen den eckigen Klammern die passende Mailserver Domain eingesetzt werden. (Bsp.: `[smtp.example.com]`).


```bash
relayhost = [smtp.example.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/postfix/cacert.pem
smtp_use_tls = yes
```

Anschließend wird die Datei mit **CTRL + X** und **yes** gespeichert.


## Erstellen von Passwort und DB Datei

Als Erstes muss eine Datei erstellt werden, welche die Anmeldedaten vom Mailserver / Postfach beinhaltet.

```bash
sudo nano /etc/postfix/sasl_passwd
```

In diese datei wird folgendes eingefügt (Auch hier muss der richtige Mailserver eingesetzt werden und `username:password` durch E-Mailadresse und Passwort ersetzt werden:

```bash
[smtp.example.com]:587 username:password
```

Mit dem `postmap` Befehl auf die eben erstellte Datei, wird die Hash Datenbank erstellt.

```bash
sudo postmap /etc/postfix/sasl_passwd
```

Im `/etc/postfix/` Verzeichnis liegt nun die *Credential-DB* mit den Hash-Werten. Diese werden anschließend mit den folgenden zwei Befehlen angepasst.

```bash
sudo chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
sudo chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```


## Generierung von Zertifikaten

Zum *Forwarden* von Mails muss ein Zertifikat generiert werden. Auch dieses liegt später im `/etc/postfix/` Verzeichnis.

```bash
cat /etc/ssl/certs/thawte_Primary_Root_CA.pem | sudo tee -a /etc/postfix/cacert.pem
```


## Testmail senden

Ist *Postfix* richtig konfiguriert, kann man den E-Mail Versand auch testen.

```bash
echo "Test Email message body" | mail -s "Email test subject" test@example.com
```

Außerdem ist es möglich Troubleshooting zu betreiben, wenn Mails nicht entsprechend ankommen.Hierzu werden die folgenden Befehle verwendet.

```bash
sudo tail /var/log/mail.log
sudo tail -f -n 50 /var/log/syslog | grep postfix
```


