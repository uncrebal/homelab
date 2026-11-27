# Wireshark-Lab

Zwei Ubuntu-Clients in einem eigenen Netz, um Protokolle wie ARP, DHCP, DNS und TCP mitzuschneiden.
Mitgeschnitten wird nur im eigenen Labor. Fremden Verkehr ohne Erlaubnis abzugreifen kann strafbar sein.

## Aufbau
```
Außennetz -- OPNsense (10.0.60.254) -- vmbr6 -- 2 Ubuntu-Clients (DHCP 10.0.60.100-200)
```

OPNsense liefert Gateway, DHCP, DNS und NAT. Ohne diese Dienste fehlt ein Teil des Verkehrs, den man untersuchen
will.

## Schritte
1. Bridge `vmbr6` ohne physischen Port anlegen.
2. OPNsense eine weitere Netzwerkkarte geben, Schnittstelle zuweisen und aktivieren.
3. DHCP-Bereich in Dnsmasq anlegen und die Schnittstelle auch unter *General* eintragen.
4. Firewall-Regeln: DHCP (UDP 68 → 67, Quelle *any*) und Verkehr aus dem Lab-Netz. Neue Schnittstellen blockieren
   sonst alles.
5. Auf den Clients `sudo apt install -y wireshark`, den Benutzer mit `sudo usermod -aG wireshark $USER` in die
   Gruppe aufnehmen und neu anmelden.

## Übungen
| Thema | Vorgehen | Filter |
|---|---|---|
| TCP-Handshake, HTTP im Klartext | `python3 -m http.server 8000`, vom anderen Client abrufen | `tcp.port == 8000` |
| ARP | `ip neigh` | `arp` |
| Neue TCP-Verbindungen | beliebigen Dienst aufrufen | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| Paketverlust | `sudo tc qdisc add dev ens18 root netem delay 200ms loss 20%` | `tcp.analysis.retransmission` |

Die Störung wieder entfernen: `sudo tc qdisc del dev ens18 root`

Capture-Filter (`port 53`) wirken vor dem Aufzeichnen, Anzeigefilter (`dns`) blenden nachträglich nur aus.

Die Fehlersuche beim Aufbau steht unter [Probleme und Lösungen](probleme-und-loesungen.md).
