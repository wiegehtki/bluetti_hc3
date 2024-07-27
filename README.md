Ich wollte unbedingt meinen Bluetti EP 500 Pro in Fibaro HC3 integrieren. Leider hat Bluetti, soweit ich sehen kann, keine direkt zugängliche lokale API. Daher musste ich ein wenig kreativ werden. Und so funktioniert es:

**Schritt 1:** Du benötigst zunächst ein Zwischenstück zwischen der Bluetti und Fibaro. Dafür verwende ich einen Raspberry Pi mit dem neuesten Raspian. Der RPi muss Bluetooth nutzen können und in der Nähe der Bluetti positioniert sein. Nachdem der RPi mit einem frischen OS gestartet wurde und du dich per SSH einloggen kannst, musst du einen Service installieren, der von https://pypi.org/user/chromedshark/ entwickelt wurde. Ich gehe davon aus, dass jemand, der dies umsetzt, das nötige Wissen hat, um einen RPi zu installieren. Dies ist also kein Leitfaden für absolute Anfänger, sorry dafür.

**Schritt 2: Installation des MOSQUITTO MQTT Brokers.** Es gibt gefühlt tausende Anleitungen dafür, benutze eine davon. Ich verwende diesen als Container, ist aber kein muss.

Dann passe die Konfiguration an:

`sudo nano /etc/mosquitto/conf.d/local.conf`

Füge hinzu:

**listener 1883**
**allow_anonymous true**

Und starte den Service neu.

**Schritt 3: Installation einer Python-Umgebung und deren Aktivierung**
Folgende Befehle sind dazu nötig:

`sudo apt install python3.11-venv`
`python3 -m venv Bluetti`
`source Bluetti/bin/activate`

Nun installiere die Abhängigkeiten, indem du eine requirements.txt erstellst und mit pip installierst:

`pip install -r requirements.txt`

Der Inhalt der requirements.txt sollte wie folgt aussehen; einfach mit nano oder vi erstellen:

bleak==0.14.0
asyncio-mqtt==0.16.2
crcmod==1.7
paho-mqtt==1.6.1
requests==2.25.1
dbus-fast==1.84.0
flask==3.0.3
flask-MQTT==1.2.1
paho-mqtt==1.6.1
gunicorn==22.0.0
dbus-next==0.2.3
bluetti-mqtt==0.15.0

Dann sollte der Befehl `bluetti-mqtt --scan` eine Ausgabe erzeugen und die Bluetti anzeigen. Falls nicht, überprüfe die Einstellungen und ggfs. die Versionen der Pakete.

d) Wenn das funktioniert, kann ein Service dafür erstellt werden. **Ändere Benutzername, IP und MAC u.a. entsprechend den Ergebnissen des Scans!** Die IP-Adresse ist dein MOSQUITTO MQTT Broker.

`sudo nano /etc/systemd/system/bluetti-mqtt.service`

Füge folgendes hinzu:

**[Unit]**
**Description=Bluetti MQTT**
**After=network.target**
**StartLimitIntervalSec=0**

**[Service]**
**Type=simple**
**Restart=always**
**RestartSec=30**
**TimeoutStopSec=15**
**User=uwuertz**
**WorkingDirectory=/home/uwuertz/Bluetti**
**ExecStartPre=/bin/sleep 15**
**ExecStart=/home/uwuertz/Bluetti/bin/bluetti-mqtt --broker 192.168.100.20 EC:6A:60:88:23:12 --interval 15**

**[Install]**
**WantedBy=multi-user.target**

Und installiere es:

`sudo systemctl daemon-reload`
`sudo systemctl start bluetti-mqtt.service`
`sudo systemctl status bluetti-mqtt.service`
`sudo systemctl enable bluetti-mqtt.service`

Letzter Schritt:

Installiere die angehängte QA-Datei und passe die IP-Adresse im Variablenfeld an.

Sobald SDM_ACTIVE auf true gesetzt ist, werden die Kindgeräte erstellt. Dann müssen die Icons der Sensoren und Schalter manuell angepasst werden. Ich werde das irgendwann schöner machen, wenn ich Zeit habe.

Die QA findet ihr unter https://github.com/wiegehtki/bluetti_hc3 (ich darf aktuell keine Anhänge hochladen weil noch neuer Benutzer)

Weitere Variablen:

* `INTERVALL` definiert die Abfrageintervalle in Sekunden
* `ICON_PARENT `setzt die ID des zu verwendenden Icons für das übergeordnete Gerät
* `ICON_CHILDS` setzt die IDs der zu verwendenden Icons für die Kindgeräte
* `ICON_SWITCHES` ist für zukünftige Verwendung
* `UNIT_IDS` = Derzeit immer 1, da aktuell nur 1 Bluetti unterstützt wird
* `SDM_ACTIVE` stoppt alle Prozesse, wenn false, und setzt sie fort, wenn true

Du benötigst auch den TOPIC-Eintrag, in meinem Fall ist das `bluetti/state/EP500P-123456781111`. 

Die Zahl ist die Seriennummer und muss mit deiner Bluetti-SN übereinstimmen. Du kannst auch einfach Tools wie den `MQTT Explorer` verwenden, nachdem der MQTT-Service installiert wurde, um zu sehen, wie der Eintrag lautet. 

Da wir einen Broker und einen zusätzlichen Service verwenden, gibt es kleine Verzögerungen zwischen dem Drücken eines binären Schalters und der ausgeführten Aktion.

**SEHR WICHTIG:** Bitte gehe einmal in den LUA-Code im Parent Device, füge irgendwo wo es unschädlich ist ein Leerzeichen ein und speichere. Vielleicht ist das ein Bug, aber ansonsten werden die binären Schalter nicht korrekt verarbeitet. Vielleicht kann jemand von Fibaro sich das mal ansehen.

**Hinweis:** Verwendung auf eigenes Risiko, ich habe die QA für meine eigenen Zwecke geschrieben und der Code kann ohne jegliche Garantie auf Vollständigkeit und Richtigkeit verwendet werden. Wenn du kein Risiko eingehen willst, lass es einfach. Der Code kann sich noch ändern. Du musst selbst testen, welche Bluettis funktionieren. Soweit ich höre, funktioniert es bei vielen Modellen. Aber keine Garantie.

Viel Spaß!
![Bluetti_Shot|690x161](upload://plaR3Ijmxv5wggaPTZDrdPaTOhg.png)
