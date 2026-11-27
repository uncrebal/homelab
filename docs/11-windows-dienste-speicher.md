# Windows-Dienste: DHCP, NAT, Drucker, NAS, iSCSI, Backup

Adressen sind Beispielwerte.

## DHCP
Rolle installieren und im AD autorisieren, Bereich anlegen, Optionen 3 (Router), 6 (DNS) und den Domänennamen
setzen. Der Dienst bindet nur an die Karte im Domänennetz. Befehle: [Windows-Domäne](07-windows-domaene.md).
Reservierungen per MAC-Adresse für die Clients sind noch nicht eingerichtet.

## NAT auf dem Server
Damit die Clients im Domänennetz ins Internet kommen:

```powershell
Install-WindowsFeature Routing -IncludeManagementTools
```

Danach in *Routing und RAS* NAT einrichten: eine Schnittstelle als öffentlich, die Domänenkarte als privat.
War im Assistenten die öffentliche Schnittstelle ausgegraut, half die benutzerdefinierte Konfiguration.
Die Domänenkarte bekommt kein Standardgateway.

In produktiven Umgebungen gehören Routing und NAT nicht auf einen Domänencontroller, sondern auf einen Router
oder eine Firewall.

| Symptom | Bedeutung |
|---|---|
| `tracert` meldet Fehler 1231 | Client hat kein Standardgateway |
| „Zielhost nicht erreichbar“ von der eigenen Adresse | Client erreicht sein Gateway nicht |
| „Zielhost nicht erreichbar“ vom Server | Server findet das Ziel nicht |

## Druckserver
Drucker zentral auf dem Server freigegeben und für die Clients bereitgestellt.

## NAS einbinden
Das NAS war nicht Mitglied der Domäne. Der Zugriff lief über einen eigenen NAS-Benutzer:

- Laufwerkzuordnung per GPO
- Zusätzlich einmalig pro Benutzer die Zugangsdaten speichern (`cmdkey /add:<NAS> /user:<NAS>\<Benutzer> /pass`,
  das Kennwort wird abgefragt)
- Ohne gespeicherte Zugangsdaten erschien das Laufwerk ohne Fehlermeldung nicht
- Kein Single Sign-on

## iSCSI: NAS-Speicher als Platte des Servers
Ziel: Clients sollen NAS-Speicher nutzen, ohne das NAS selbst zu erreichen.

```mermaid
flowchart LR
    N[NAS: Target und LUN] -- iSCSI --> D[Server: lokale Platte, NTFS]
    D -- SMB-Freigabe --> C[Clients]
```

| Begriff | Bedeutung |
|---|---|
| Target | bietet Speicher an (NAS) |
| Initiator | verbindet sich (Server) |
| LUN | der Speicherbereich, für den Server wie eine Platte |
| IQN | eindeutiger Name von Target und Initiator |

Grenzen: Eine LUN darf ohne Cluster-Dateisystem nur von einem Rechner benutzt werden. iSCSI ersetzt kein
Backup. Der Test lief ohne CHAP-Anmeldung und ohne Verschlüsselung, für den Dauerbetrieb müsste beides bewertet
werden.

Stand: Verbindung, Platte und Freigabe auf dem Server sind eingerichtet. Der Test vom Client ist noch nicht
dokumentiert.

## Windows Server-Sicherung
- Gesichert werden Bare-Metal-Recovery (mit Systemstatus inklusive AD) und die Datenplatte
- Geplante Sicherungen auf eine Freigabe brauchen ein Konto, unter dem die Aufgabe auf dem Server laufen darf und
  das Zugriff auf das Ziel hat. Dafür möglichst ein eigenes Sicherungskonto in den *Sicherungs-Operatoren*.
- Einschränkung: Auf einer Netzwerkfreigabe bleibt nur die letzte Sicherung erhalten. Snapshots auf dem NAS
  wären die Ergänzung.

Stand: Der Zeitplan ist eingerichtet. Eine geprüfte Wiederherstellung ist noch nicht dokumentiert.
