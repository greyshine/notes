Ich habe eine Domain bei Strato und möchte via Fritzbox als Router auf einen weiteren Rechner in meinem Netzwerk zugreifen.

Ich komme aus dem öffentlichen Internet jedoch nicht durch.
Im lokalen Netz komme ich auch unter Eingabe der Domain `https://sub.example.de` durch. 
Mein Glasfaserprovider ist Westconnect (EON).

Ich habe dies zum Testen auf Port `4443`, weil mir der Port `443` auf dem Linux-Server nicht erlaubt wird. 

Folgende Beiträge schon angeuckt hier:

- https://www.reddit.com/r/fritzbox/comments/1ij85bb/dyndns_port_freigaben
- https://www.reddit.com/r/fritzbox/comments/1f40a35/dyndns_not_working

Szenario wie folgt:

# Domain Provider - Strato
https://sub.example.de

- DynDNS ist aktiviert.

# Fritzbox

Fritz!Box 7590, FRITZ!OS 8.02

- `Internet/Online-Monitor/Verbindungsdetails/Internet-Verbindung` zeigt, dass IPv4 und IPv6 verbunden sind. 

- Internet/Freigaben
  
  __IPv6-Einstellungen__  
  check `Ping6 freigeben`  
  check `Firewall für delegierte IPv6-Präfixe dieses Gerätes öffnen.`  
  uncheck `Dieses Gerät komplett für den Internetzugriff über IPv6 freigeben (Exposed Host).`
  
  `Port 80 -> 192.168.172.10:10000`
  `Port 443 -> 192.168.172.10:10001`

- DynDNS aktiviert und eingetragen

  Bei `Internet/Freigaben/DynDNS`
  
  Update-URL: `https://<username>:<passwd>@dyndns.strato.com/nic/update?hostname=<domain>&myip=<ipaddr>,<ip6addr>`  
  Domainname: `sub.example.de`  
  Benutzername: `example.de`
  Kennwort: `****`

  Unter `Internet/Online-Monitor/Verbindungsdetails`
  steht bei DynDNS: ` DynDNS aktiv, sub.example.de, IPv4-Status: angemeldet, IPv6-Status: angemeldet`

- IP4 vs IP6 Sachverhalt
  
  Ich bin bei dem Glasfaseranbieter Westconnect und hatte gelesen, dass die die IP4-Adressen bündeln, was man an dem Beginn der IP4-Adresse sehen kann `100.68.x.y`. (`Internet/Online-Monitor/Verbindungsdetails`).
  Daraus schlussfolgernd muss ich komplett auf IPv6 setzen, da eine 1:1 Identifizierung meines Anschluss nicht via IPv4 geht und nur über IPv6.

  (Weiss nicht mehr wo ich es gelesen habe.)

- DNS-Rebind-Schutz

  Bei `Heimnetz/Netzwerk/Netzwerkeinstellungen` folgendes in der Texteingabebox:
  ```
  example.de
  sub.example.de
  ```

  Hatte ich in meinen Zahlreichen versuchen von der KI aufgeschnappt.

# Ubuntu-Server - 192.168.178.10
  
  Server im Heimnetz.

  - Firewall ist offen für `ip4` und `ip6` mit `ufw` für `TCP` und `UDP` für die Ports `80, 443, 10000, 10001`

  ```
  443/tcp                    ALLOW       Anywhere
  443/udp                    ALLOW       Anywhere
  ...
  443/tcp (v6)               ALLOW       Anywhere (v6)
  443/udp (v6)               ALLOW       Anywhere (v6)
  ...
  ```

  Meine Test-Web-Server laufen auf `HTTP` Port `10000` und `HTTPS` Port `10001`.
  Sie sind im Heinmentz ansprechbar und liefern ordnungsgemäß ihren Content aus.  
  Der Https ist nicht signiert und meldet dies auch im Browser; Anbindung an Let's Encrypt kommt später.

  Erlaubnis des Servers auf `443` zu laufen hatte ich mit `sudo setcap 'CAP_NET_BIND_SERVICE+ep' /usr/bin/python3.12` erteilt.

# Was wie aus dem öffentlichen Netz geht bzw. nicht geht.

- `ping` auf `sub.example.de` geht durch.
Scheint mir am IP6 Sachverhalt zu liegen, weil mein Glasfaseranbieter Zugänge bündelt.

- `ping6` auf `sub.example.de` geht nicht durch.

- Aufruf auf `https://sub.example.de` via Internet geht nicht durch.

- Aufruf auf `https://sub.example.de` im lokalen Netz geht.

# Resumée

Ich gehe davon aus, dass noch irgendetwas an der Fritzbox anders eingestellt werden muss, aber ich weiss nicht was...  
Kann mir hier jemand helfen und mir zeigen, was ich noch nicht verstanden habe?
