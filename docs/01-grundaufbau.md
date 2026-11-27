# Grundaufbau

## Installation
1. Im BIOS VT-x und VT-d einschalten (VT-d für das Passthrough).
2. Proxmox VE installieren, statische IP vergeben (Beispiel `10.0.10.10/24`).
3. Ohne Abo: Enterprise-Repositories deaktivieren, *No-Subscription* hinzufügen.
4. `apt update && apt full-upgrade` (bei Proxmox `full-upgrade`, nicht nur `upgrade`).

## Speicher
Zusätzliche Datenträger sind als eigene LVM-Thin-Pools eingebunden. Vorher mit SMART prüfen:

```bash
apt install -y smartmontools
smartctl -H /dev/sda
```

## Bridges

| Bridge | Zweck |
|---|---|
| `vmbr0` | Außennetz, einzige Bridge mit physischem Port |
| `vmbr1` | Domänennetz |
| `vmbr2` | Labornetz hinter OPNsense |
| `vmbr3`–`vmbr5` | Netze des Routing-Labors |
| `vmbr6` | Analyse-Lab |

## Container oder VM?
- LXC-Container: einzelne schlanke Linux-Dienste (Datenbank, Paket-Cache, VPN, Zabbix).
- VM: Windows, Firewall, Router, Desktop-Systeme, Docker-Host.

## Paket-Cache
Ein Container mit `apt-cacher-ng` speichert Linux-Pakete zwischen. Bei einer langsamen Leitung muss ein Paket so
nur einmal aus dem Internet geladen werden. HTTPS-Quellen werden nicht zwischengespeichert.

## Vorlagen und Klone
Statt jedes System neu zu installieren, habe ich Vorlagen (Templates) angelegt und daraus neue VMs geklont:

| Vorlage | Daraus entstanden |
|---|---|
| Ubuntu Server | Docker-VM, beide Linux-Router, Linux-Server im Routing-Labor |
| Ubuntu Desktop | zwei Clients für das Wireshark-Lab |
| Debian Desktop | Debian-Client für die Übungen zur Absicherung |
| Windows 11 (mit Sysprep) | vier Windows-Clients für die Domäne |

In Proxmox: VM fertig einrichten, herunterfahren, Rechtsklick → *Convert to template*. Neue VM über Rechtsklick auf
die Vorlage → *Clone*.

- **Full Clone:** eine komplett unabhängige Kopie. So habe ich die meisten VMs erstellt.
- **Linked Clone:** speichert nur die Unterschiede zur Vorlage, ist in Sekunden fertig und braucht kaum Platz, hängt
  aber von der Vorlage ab.

Dabei gelernt: Mehrere Klone gleichzeitig auf dieselbe Festplatte haben die Weboberfläche lahmgelegt. Seitdem klone
ich nacheinander.

### Linux-Vorlagen
Vor dem Umwandeln in ein Template die Machine-ID leeren, sonst erhalten die Klone beim DHCP dieselbe Adresse:

```bash
sudo truncate -s 0 /etc/machine-id && sudo poweroff
```

Bei geklonten Servern zusätzlich prüfen, dass jedes System eigene SSH-Host-Schlüssel hat.

## Routing-Labor
Zwei Linux-Router verbinden drei getrennte Netze:

```
Server --LAN1-- Router1 --LAN0-- Router2 --LAN2-- Windows-Client
```

Die Linux-VMs haben zusätzlich eine Karte nur für die Verwaltung per SSH. Welche Karte an welcher Bridge hängt,
zeigt der Vergleich der MAC-Adressen aus Proxmox mit `ip -br link`. Adressplan und Konfiguration:
[Linux-Routing](13-linux-routing.md).
