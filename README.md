# CCU-Jack

CCU-Jack bietet einen einfachen und sicheren REST-basierten Zugriff auf die Datenpunkte der Zentrale (CCU) des [Hausautomations-Systems](http://de.wikipedia.org/wiki/Hausautomation) HomeMatic der Firma [eQ-3](http://www.eq-3.de/). Er implementiert dafür das [Very Easy Automation Protocol](https://github.com/mdzio/veap), welches von vielen Programmiersprachen leicht verwendet werden kann.

## Hauptmerkmale

Folgende Merkmale zeichnen CCU-Jack aus:
* Lese- und Schreibzugriff auf alle Gerätedatenpunkte und Systemvariablen der CCU.
* Alle Datenpunkte können über eine Baumstruktur erkundet werden.
* Umfangreiche Zusatzinformationen zu jedem Datenpunkt, z.B. Anzeigenamen, Räume, Gewerke, aber auch viele technische Informationen aus den XMLRPC-Schnittstellen und der ReGaHss.
* Hohe Performance und minimale Belastung der CCU-Prozesse (XMLRPC-Schnittstellen, ReGaHss, CCU Web-Server).
* Unterstützung von HTTP/2 und Verbindungssicherheit auf dem Stand der Technik.
* Fertige Distributionen für viele Zielsysteme (CCU2, CCU3/RM, Windows, Linux, macOS).
* Die Verwendung des VEAP-Protokolls ermöglicht einfachste Anbindung von Applikationen und Frameworks (z.B. Angular, React, Vue). Zudem ist das Protokoll nicht CCU-spezifisch. Entwickelte Client-Applikationen könnnen auch mit anderen VEAP-Servern verwendet werden.

## Projekt

Ziel vom CCU-Jack ist es, möglichst einfach Datenpunkte zwischen CCUs und auch anderen Systemen auszutauschen und Applikationen eine Erkundungsmöglichkeit der Datenpunkte zu bieten. Der CCU-Jack wurde komplett neu entwickelt. Vorgänger vom CCU-Jack sind schon längere Zeit in Betrieb und tauschen hunderte von Datenpunkten zwischen mehreren CCUs, Internet-Relays und Front-Ends in Echtzeit aus.

Für die Version 1.0 ist noch die Funktionalität VEAP-Client geplant. Dadurch ist es möglich mehrere CCU-Jacks bzw. VEAP-Server untereinander zu verbinden. Verbindungen sollen dann einfach per Web-Browser angelegt werden können.

## Download

Distributionen für die verschiedenen Zielsysteme sind auf der Seite [Releases](https://github.com/mdzio/ccu-jack/releases) zu finden. 

Zurzeit besitzt CCU-Jack noch Beta-Status. Es sollten also vor der Verwendung Sicherheitskopien von den involvierten Systemen erstellt werden (z.B. System-Backup von der CCU).

### Installation als Add-On auf der CCU

Bei einer Installation als Add-On auf der CCU können die Startparameter in der Datei `/usr/local/etc/config/rc.d/ccu-jack` angepasst werden. In der Regel ist dies nicht notwendig. Log-Meldungen werden vom Add-On nicht ausgegeben oder gespeichert. Bei Bedarf kann die genannte Datei aber abgeändert werden, sodass die Log-Meldungen in eine Datei geschrieben werden.

In der Firewall der CCU müssen die zwei Ports 2121 und 2122 freigegeben werden:
![CCU-Firewall](doc/ccu-firewall.png)

## Kommandozeilenoptionen

Die Kommandozeilenoptionen vom CCU-Jack werden beim Start mit der Option `-h` aufgelistet:
```
usage of ccu-jack:
  -addr address
        address of the host (default "127.0.0.1")
  -ccu address
        address of the CCU (default "127.0.0.1")
  -host name
        host name for certificate generation (normally autodetected)
  -id identifier
        additional identifier for the XMLRPC init method (default "CCU-Jack")
  -interfaces types
        types of the CCU communication interfaces (comma separated): BidCosWired, BidCosRF, System, HmIPRF, VirtualDevices (default BidCosRF)
  -log severity
        specifies the minimum severity of printed log messages: off, error, warning, info, debug or trace (default INFO)
  -password password
        password for HTTP Basic Authentication, q.v. -user
  -port port
        port for serving HTTP (default 2121)
  -tls port
        port for serving HTTPS (default 2122)
  -user name
        user name for HTTP Basic Authentication (disabled by default)
```

Log-Meldungen werden auf der Fehlerausgabe (STDERR) ausgegeben, wenn sie mindestens die mit der Option `-log` gesetzte Dringlichkeit besitzen.

## Performance

Folgende Angaben gelten für eine Installation als Add-On auf einer CCU3 (Raspberry Pi 3B):
* 1,7 Millisekunden Latenz für das Lesen eines Datenpunktes.
* 8.800 Datenpunkte können von 100 Clients pro Sekunde gesichert mit HTTPS-Verschlüsselung gelesen werden.

## Web-basierte Benutzerschnittelle

Die web-basierte Benutzerschnittstelle des CCU-Jacks ist über die Adressen `http://<host>:2121/ui` und `https://<host>:2122/ui` zu erreichen. `<host>` ist durch den Rechnernamen oder die IP-Adresse des Rechners zu ersetzen, auf dem CCU-Jack gestartet worden ist. Wenn es der lokale Rechner ist, kann auch `localhost` verwendet werden.

Mit Hilfe des Navigators können alle verfügbaren Datenpunkte erkundet werden:

![Navigator](doc/web-ui-navigator.png)

Bei Variablen wird auch der Wert angezeigt und aktuell gehalten:

![Navigator mit Variable](doc/web-ui-navigator-var.png)

## Beispiele für die Android App _HTTP Shortcuts_

CCU-Jack ermöglicht der kostenlosen Android App _HTTP Shortcuts_ einfachen Zugriff auf die Datenpunkte der CCU. So können beispielsweise Geräte direkt vom Home-Screen geschaltet werden. Beispiele sind auf einer [eigenen Seite](https://github.com/mdzio/ccu-jack/blob/master/doc/httpshortcuts.md) zu finden.

## CURL-Beispiele

Im Folgenden werden einige Beispielaufrufe mit dem [Werkzeug CURL](https://curl.haxx.se) gezeigt. Manche Antworten wurden aus Platzgründen gekürzt.

### Datenpunkt lesen

Das Lesen eines Datenpunktes erfolgt über die HTTP-Methode GET. Der aktuelle Wert kann also auch mit einem Web-Browser gelesen werden. In der Web-UI der CCU kann unter _Einstellungen_ → _Geräte_ die Seriennummer des Kanals ermittelt werden. Der `:` in der Seriennummer muss duch `/` ersetzt werden. Der Parametername ist beispielsweise  _STATE_ bei einem Schaltaktor oder _LEVEL_ bei einem Rollladenaktor.

Aufruf:
```
curl http://localhost:2121/device/HEQ0123456/1/STATE/~pv
```
Antwort:
```json
{
  "ts": 1546297200000,
  "v": false,
  "s": 0
}
```

Die Eigenschaft `v` enthält den aktuellen Wert des Datenpunktes. `ts` enthält den Zeitstempel der letzten Aktualisierung des Wertes (Millisekunden seit 1.1.1970 UTC, hier 1.1.2019 00:00 MEZ). `s` gibt Auskunft über die Qualität des Wertes (hier immer 0 → Gut).

### Datenpunkt setzen

Das Setzen eines Datenpunktes erfolgt über die HTTP-Methode PUT.

Aufruf (Linux):
```
curl -X PUT -d '{"v":true}' http://localhost:2121/device/HEQ0123456/1/STATE/~pv
```
Aufruf (Windows):
```
curl -X PUT -d "{""v"":true}" http://localhost:2121/device/HEQ0123456/1/STATE/~pv
```
Antwort: 
```
HTTP/1.1 200 OK
```

Fehler werden über den HTTP-Status angezeigt.

### Erkundung von / (Wurzelverzeichnis)

Aufruf:
```
curl http://127.0.0.1:2121
```
Antwort:
```json
{
  "description": "Root of the CCU-Jack VEAP server",
  "identifier": "root",
  "title": "Root",
  "~links": [
    {
      "rel": "domain",
      "href": "~vendor",
      "title": "Vendor Information"
    },
    {
      "rel": "domain",
      "href": "device",
      "title": "Devices"
    },
    {
      "rel": "domain",
      "href": "sysvar",
      "title": "System variables"
    }
  ]
}
```

Die Eigenschaft `href` der Objekte im `~links`-Array kann zur weiteren Erkundung des VEAP-Servers genutzt werden.

### Erkundung von /device (Geräte)

```
curl http://127.0.0.1:2121/device
```
Antwort:
```json
{
  "description": "CCU Devices",
  "identifier": "device",
  "title": "Devices",
  "~links": [
    {
      "rel": "device",
      "href": "NEQ1234567",
      "title": "Rollladen Bad"
    },
    {
      "rel": "device",
      "href": "OEQ0123456",
      "title": "Türkontakt Terrasse"
    },
    {
      "rel": "device",
      "href": "GEQ0123456",
      "title": "Bewegungsmelder Flur EG"
    },
    ...
  ]
}
```

### Erkundung von /device/OEQ0123456 (Türkontakt Terrasse)

```
curl http://localhost:2121/device/OEQ0123456
```
Antwort:
```json
{
  "address": "OEQ0123456",
  "aesActive": 0,
  "availableFirmware": "",
  "children": [
    "OEQ0123456:0",
    "OEQ0123456:1"
  ],
  "direction": 0,
  "firmware": "2.4",
  "flags": 1,
  "group": "",
  "identifier": "OEQ0123456",
  "index": 0,
  "interface": "PEQ0123456",
  "linkSourceRoles": "",
  "linkTargetRoles": "",
  "paramsets": [
    "MASTER"
  ],
  "parent": "",
  "parentType": "",
  "rfAddress": 5855058,
  "roaming": 0,
  "rxMode": 12,
  "team": "",
  "teamChannels": null,
  "teamTag": "",
  "title": "Türkontakt Terrasse",
  "type": "HM-Sec-SC-2",
  "version": 16,
  "~links": [
    {
      "rel": "channel",
      "href": "0",
      "title": "Türkontakt Terrasse:0"
    },
    {
      "rel": "channel",
      "href": "1",
      "title": "Türzustand Terrasse"
    },
    {
      "rel": "devices",
      "href": "..",
      "title": "Devices"
    }
  ]
}
```

### Erkundung von /device/OEQ0123456/1 (Kanal 1)

```
curl http://localhost:2121/device/OEQ0123456/1
```
Antwort:
```json
{

    "address": "OEQ0123456:1",
    "aesActive": 1,
    "availableFirmware": "",
    "children": null,
    "direction": 1,
    "firmware": "",
    "flags": 1,
    "group": "",
    "identifier": "1",
    "index": 1,
    "interface": "",
    "linkSourceRoles": "KEYMATIC SWITCH WINDOW_SWITCH_RECEIVER WINMATIC",
    "linkTargetRoles": "",
    "paramsets": [
        "LINK",
        "MASTER",
        "VALUES"
    ],
    "parent": "OEQ0123456",
    "parentType": "HM-Sec-SC-2",
    "rfAddress": 0,
    "roaming": 0,
    "rxMode": 0,
    "team": "",
    "teamChannels": null,
    "teamTag": "",
    "title": "Türzustand Terrasse",
    "type": "SHUTTER_CONTACT",
    "version": 16,
    "~links": [
        {
            "rel": "parameter",
            "href": "INSTALL_TEST",
            "title": "INSTALL_TEST"
        },
        {
            "rel": "parameter",
            "href": "LOWBAT",
            "title": "LOWBAT"
        },
        {
            "rel": "parameter",
            "href": "STATE",
            "title": "STATE"
        },
        {
            "rel": "parameter",
            "href": "ERROR",
            "title": "ERROR"
        },
        {
            "rel": "device",
            "href": "..",
            "title": "Türkontakt Terrasse"
        },
        {
            "rel": "room",
            "href": "/room/1248",
            "title": "Wohnzimmer"
        },
        {
            "rel": "function",
            "href": "/function/1243",
            "title": "Sicherheit"
        }
    ]
}
```

Bei den `~links` sind ebenfalls Verweise auf die Räume (`"rel":"room"`) und Gewerke (`"rel":"function"`) des Kanals zu finden. In den `title`-Eigenschaften sind in der Regel die benutzerspezifischen Namen zu finden. Die Abfrageadresse eines Datenpuntkes ändert sich also nicht, wenn der Kanal in der CCU umbenannt wird. Das gleiche gilt auch für Systemvariablen.

### Erkundung von /device/OEQ0123456/1/STATE (Parameter STATE)

```
curl http://localhost:2121/device/OEQ0123456/1/STATE
```
Antwort:
```json
{
  "control": "DOOR_SENSOR.STATE",
  "default": false,
  "flags": 1,
  "id": "STATE",
  "identifier": "STATE",
  "maximum": true,
  "minimum": false,
  "operations": 5,
  "tabOrder": 0,
  "title": "STATE",
  "type": "BOOL",
  "unit": "",
  "~links": [
    {
      "rel": "channel",
      "href": "..",
      "title": "Türzustand Terrasse"
    },
    {
      "rel": "service",
      "href": "~pv",
      "title": "PV Service"
    }
  ]
}
```

Der Dienst `PV Service` kennzeichnet einen Datenpunkt. Durch Anhängen von `/~pv` an die Adresse des Datenpunktes kann auf den aktuellen Wert zugegriffen werden.

### Erkundung von /room (Räume)

```
curl http://localhost:2121/room
```
Antwort:
```json
{
    "description": "Rooms of the ReGaHss",
    "identifier": "room",
    "title": "Rooms",
    "~links": [
        {
            "rel": "room",
            "href": "1248",
            "title": "Wohnzimmer"
        },
        {
            "rel": "room",
            "href": "2194",
            "title": "Außengelände"
        },
        {
            "rel": "room",
            "href": "6510",
            "title": "Bad"
        },
        ...
    ]
}
```

Das oben genannte gilt analog für die Erkundung der Gewerke (`/function`).

### Erkundung von /room/123 (Raum)

```
curl http://localhost:2121/room/1248
```
Antwort:
```json
{
    "identifier": "1248",
    "title": "Wohnzimmer",
    "~links": [
        {
            "rel": "collection",
            "href": "..",
            "title": "Rooms"
        },
        {
            "rel": "channel",
            "href": "/device/GEQ0123456/1",
            "title": "Bewegung Wohnzimmer"
        },
        {
            "rel": "channel",
            "href": "/device/GEQ0012345/1",
            "title": "Rollladen Esstisch"
        },
        ...
    ]
}
```

In der `~links`-Eigenschaft werden die einem Raum zugewiesenen Kanäle aufgelistet. Über den enthaltenen Verweis kann direkt zu den Datenpunkten gesprungen werden.

Das Gleiche gilt analog für ein Gewerk.

### Erkundung von /~vendor (Herstellerinformationen)

```
curl http://127.0.0.1:2121
```
Antwort:
```json
{
  "description": "Information about the server and the vendor",
  "identifier": "~vendor",
  "serverDescription": "VEAP-Server for the HomeMatic CCU",
  "serverName": "CCU-Jack",
  "serverVersion": "1.0.0-alpha.2",
  "title": "Vendor Information",
  "veapVersion": "1",
  "vendorName": "info@ccu-historian.de",
  "~links": [
    {
      "rel": "item",
      "href": "statistics",
      "title": "HTTP(S) Handler Statistics"
    },
    {
      "rel": "collection",
      "href": "..",
      "title": "Root"
    }
  ]
}
```

## Sicherer Zugriff über HTTPS

CCU-Jack ermöglicht einen verschlüsselten Zugriff über HTTPS, sodass auch über unsichere Netzwerke (z.B. Internet) Daten sicher ausgetauscht werden könnan. Über den Port 2122 (änderbar mit der Kommandozeilenoption `-porttls`) kann eine HTTPS-Verbindung aufgebaut werden. Die dafür benötigten Zertifikate können vorgegeben werden oder werden beim ersten Start vom CCU-Jack automatisch generiert.

Benötigte Zertifikatsdateien für den Server:

Dateiname   | Funktion
------------|---------
svrcert.pem | Zertifikat des Servers
svrcert.key | Privater Schlüssel des Servers (Dieser ist geheim zu halten.)

Falls die oben genannten Zertifikatsdateien im Arbeitsverzeichnis des CCU-Jacks nicht vorhanden sind, so werden automatisch zwei Zertifikate erstellt. Die Gültigkeit ist auf 10 Jahre eingestellt:

Dateiname   | Funktion
------------|---------
cacert.pem  | Zertifikat der Zertifizierungsstelle (CA)
cacert.key  | Privater Schlüssel der Zertifizierungsstelle (Dieser ist geheim zu halten.)
svrcert.pem | Zertifikat des Servers
svrcert.key | Privater Schlüssel des Servers (Dieser ist geheim zu halten.)

Für den sicheren Zugriff muss lediglich das generierte Zertifikat der Zertifizierungsstelle (`cacert.pem`) den HTTPS-Clients *über einen sicheren Kanal* bekannt gemacht werden. Das Zertifikat kann z.B. im Betriebssystem oder im Web-Browser installiert werden. Die privaten Schlüssel dürfen nie verteilt werden.

Über verschiedene Programmiersprachen kann auch verschlüsselt zugegriffen werden.

### Curl

```bash
curl --cacert path/to/cacert.pem https://hostname:2122
```

### Python

```python
import requests
r = requests.get("https://hostname:2122", verify='path/to/cacert.pem')
print(r.status_code)
```

### Go

```go
caCert, err := ioutil.ReadFile("path/to/cacert.pem")
if err != nil {
    log.Fatal(err)
}
caCerts := x509.NewCertPool()
ok := caCerts.AppendCertsFromPEM(caCert)
if !ok {
    log.Fatal("Failed to parse certificate")
}
con, err := tls.Dial("tcp", "hostname:2122", &tls.Config{RootCAs: caCerts})
if err != nil {
    log.Fatal(err)
}
defer con.Close()
```


### Javascript

```javascript
var fs = require('fs');
var https = require('https');

var get = https.request({
  path: '/', hostname: 'hostname', port: 2122,
  ca: fs.readFileSync('path/to/cacert.pem'),
  agent: false,
  rejectUnauthorized: true,
}, function(response) {
  response.on('data', (d) => {
    process.stdout.write(d);
  });
});
get.on('error', function(e) {
  console.error(e)
});
get.end();
```

## Lizenz und Haftungsausschluss

Lizenz und Haftungsausschluss sind in der Datei [LICENSE.txt](LICENSE.txt) zu finden.