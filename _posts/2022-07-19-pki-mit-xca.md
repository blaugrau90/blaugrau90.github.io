---
title: Eigene PKI mit dem Tool XCA
date: 2022-07-19 13:00:00 +200
categories: [Homelab, Security, SSL, Zertifikate]
tags: [xca,pki,ssl,security]
comments: true
---

Für die Erstellung einer eigenen PKI (Publik Key Infrastructure) wird die Anwendung XCA auf der [Herstellerseite](https://hohnstaedt.de/xca/) heruntergeladen und installiert. Diese Anwendung XCA ist für die Erstellung und Verwaltung von X.509-Zertifikaten, Zertifikatsanträgen, privaten RSA-, DSA- und EC-Schlüsseln, Smartcards und CRLs vorgesehen. Alle CAs können rekursiv Sub-CAs signieren. Diese Zertifikatsketten werden übersichtlich dargestellt. Für einen einfachen unternehmensweiten Einsatz gibt es anpassbare Vorlagen, die für die Zertifikats- oder Request-Erstellung verwendet werden können.

## Hinweise zum Aufbau einer Unternehmensstruktur

In einem Unternehmen werden häufig eine Root CA und entsprechende Sub CAs erstellt, welche dann Server und auch Client-Zertifikate ausstellen und signieren können.

Der Aufbau einer solchen Struktur kann mit XCA ebenfalls abgebildet werden. XCA kann wie folgt verwendet werden.

Ein korrekter Aufbau ist deshalb notwendig, um die sog. Chain of Trust sicherzustellen, also die Vertrauenswürdigkeit der gesamten Zertifikatskette von Server-Zertifikat bis zurück zum Root Zertifikat. Der Aufbau und das Signing wird im folgenden Bild beschrieben (<https://en.wikipedia.org/wiki/Chain_of_trust#/media/File:Chain_Of_Trust.svg>)


 ![](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/01.png?raw=true)


## Erstellung der Root CA

Um eine solche Struktur einzurichten beginnen wir in XCA mit dem erstellen einer Root CA. Dazu wird die Anwendung zunächst geöffnet und eine neue Datenbank erstellt. (Datei → Neue Datenbank) 

Die Datenbank sollte mit einem sicheren Passwort versehen werden, sodass der unbefugte Zugriff ausgeschlossen ist. 

### Erstellen des Root Zertifikats

Um das Root Zertifikat zu erstellen wird im Tab *Zertifikate* über die Schaltfläche *Neues Zertifikat* ein neues Zertifikat generiert.

Im Dialog wählt man hier zuerst die Default Vorlage für eine *CA* aus und klickt auf *Alles übernehmen*. Damit werden die Standart-Settings für ein Root CA Zertifikat gesetzt.

 ![Erstellung eines neuen, selbstsignierten Zertifikats für die Root CA](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/02.png?raw=true)

Mit dem Wechsel auf den Tab *Inhaber* kommt man zu den weiteren Einstellungen im Zertifikat. Hier sollten alle Felder wie im unten stehenden Screenshot ausgefüllt werden. Wichtig hierbei ist vor allem der *Common Name*. Dieser enthält den FQDN (Fully Qualified Domain Name) des Servers. In diesem Fall haben wir diesen Namen jedoch noch nicht und tragen daher den Namen der Root CA ein. Bevor man weiter zum nächsten Tab wechselt wird noch ein neuer Key für die CA erstellt. Dieser sollte 4096 Bit lang sein. 

 ![Erstellung des Keys für die Root CA](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/03.png?raw=true)

 ![Einstellungen zum Inhaber](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/04.png?raw=true)


Der Tab Erweiterungen kann so belassen werden. Jedoch sollte man sich hier über die Zeitspanne der Gültigkeit des Zertifikats Gedanken machen. Hier sind default 10 Jahre eingetragen. In der Regel sind Root CAs jedoch 15-30 Jahre gültig, da ein schneller Wechsel von Root Zertifikaten kaum, oder nur mit großem Aufwand möglich ist. Daher sollte der Wert etwas angepasst / angehoben werden. Der Zeitraum wird erst mit dem Klick auf *Übernehmen* auch wirklich übernommen. 

 ![Zertifikatsgültigkeit](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/05.png?raw=true)


Alle weiteren Tabs sind bereits richtig vorkonfiguriert und müssen nicht weiter bearbeitet werden. Mit dem abschließenden Klick auf *OK* wird das Zertifikat ausgestellt und ist ab dann gültig.

 ![Ausgestelltes Root CA](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/06.png?raw=true)


## Erstellung einer Sub CA - Intermediate Certificate

Ein Intermediate Certificate, oder auch Zwischenzertifikat, wird dazu verwendet, um dann später Webserver- oder Client-Zertifikate auszustellen. Das Zertifikat der SubCA wird von der Root CA unterschrieben. Der Vorgang der Ausstellung ist also sehr ähnlich, weswegen hier das Root CA als Vorlage dienen kann.

### Erstellen des Sub Zertifikats

Über einen einfachen Rechtsklick auf das Root Zertifikat, kann ein neues, ähnliches Zertifikat generiert werden.

 ![Erstellen eines Ähnlichen Zertifikats](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/07.png?raw=true)


Es öffnet sich anschließend wieder der Ausstellungsdialog. Dort wählt man nun zunächst die Root CA als unterschreibende Zertifizierungsstelle aus und wechselt dann zum Tab Inhaber. Eine Vorlage muss nicht angewendet werden, da wir das Root Zertifikat bereits als Vorlage verwenden.

 ![Root CA als Signaturzertifikat auswählen](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/08.png?raw=true)


Im *Inhaber* Tab wird dann noch der Interne und der Common Name angepasst. Hier bietet es sich an, die Sub CA eventuell direkt zu spezifizieren und eine Versionsnummer mit anzugeben. Auch hier ist es wichtig, wieder einen neuen Schlüssel mit 4096 Bit zu generieren.

 ![](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/09.png?raw=true)

Im Tab Erweiterungen muss die Gültigkeitsdauer ebenfalls entsprechend geändert werden. Es empfiehlt sich, hier einen kleineren Zeitraum als den von der Root CA zu verwenden. In der Regel die Halbe Zeit der Root CA. 

 ![](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/10.png?raw=true)

Mit *Übernehmen* und *OK* wird auch dieses Zertifikat erstellt und ist nun in der Liste sichtbar. Deutlich wird hier bereits, dass eine Zertifikatskette entstanden ist, welche aufeinander aufbaut.

 ![Zertifikatskette von Root und Sub CA](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/11.png?raw=true)


## Erstellen von Server Zertifikaten

Nach der Erstellung der Root und Sub CA, können nun entsprechende Server Zertifikate ausgestellt werden. Diese dienen für den Browser dann zur verschlüsselten Verbindung zwischen User und Server.

Die Erstellung solcher Zertifikate ist ebenso einfach wie auch für die CA Zertifikate. Lediglich die Vorlage für die Ausstellung ist eine andere. 

 ![Erstellung eines Serverzertifikats von der Sub CA unterschrieben](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/12.png?raw=true)

Im vorangegangenen Screenshot wird entsprechend *TLS_Server* als Vorlage übernommen und die Sub CA als unterschreibende CA ausgewählt. Der Inhaber Tab ist ähnlich auszufüllen wie bei den CA Zertifikaten. 

 ![](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/13.png?raw=true)

Besonders wichtig sind hier jedoch der Interne und auch der Common Name. Im CN MUSS der FQDN stehen. Der Interne Name ist frei wählbar, sollte jedoch auf den Hostname des Servers verweisen. Bei der Erstellung des Keys für dieses Zertifikat ist eine 2048 Bit Keylänge ausreichend. 

 ![](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/14.png?raw=true)


Im Tab *Erweitert* werden neben dem Gültigkeitszeitraum auch noch weitere Settings gemacht. Unter *Subject Alternative Name* sollten mindestens drei Einträge stehen. Zum einen der FQDN (dieser wird durch den bereits vorhandenen Eintrag automatisch befüllt), sowie der Hostname (DNS:hostname) und eine IP (IP:10.10.10.20).

 ![](https://github.com/blaugrau90/blaugrau90.github.io/blob/main/assets/img/postimg/2022-07-19-pki-mit-xca/15.png?raw=true)

Nach diesem Schritt kann auch hier wieder mit *OK* das Zertifikat entsprechend erstellt werden. 


## Fazit

Die Einrichtung und Konfiguration einer eigenen PKI mit dem Tool XCA ist sehr einfach und übersichtlich. Alle Zertifikate und Keys werden sicher in einer Datenbank aufbewahrt und sind mit einem Passwort geschützt. Wer sich ganz Sicher sein will, dass die Root CA nicht kompromittiert wird, erstellt jeweils für die Root CA und für die Sub CA eine eigene Datenbank und bewahrt diese getrennt voneinander und mit verschiedenen Kennwörtern auf.

Die Einrichtung von Getrennten CAs ist nicht merklich schwieriger. Hier wird nur nach dem Erstellen der Sub CA selbige mit den entsprechenden Zertifikats- und Key-Files als PKCS12 (.pfx-File) Exportiert und eine neue Datenbank erstellt. Anschließend wird die CA hier wieder importiert und kann dann verwendet werden um Server- oder Client-Zertifikate zu unterschreiben.

Zertifikate und Keyfiles lassen sich sehr einfach Exportieren und können so auf Webservern einfach wieder als File importiert werden.