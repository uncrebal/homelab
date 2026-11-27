# Windows-Netz ohne Domäne (Peer-to-Peer)

Vorstufe zum Domänenprojekt: vier Windows-11-PCs in einer Arbeitsgruppe. Namen und Adressen sind Beispielwerte.

## Inhalt
- Statische IP-Adressen mit festem Schema (Bereiche für Server, Arbeitsplätze, Drucker und Router)
- Netzwerkerkennung, Datei- und Druckerfreigabe, kennwortgeschütztes Freigeben
- Lokale Benutzer und Gruppen. Ohne zentrale Anmeldung muss ein Konto auf jedem PC mit gleichem Namen und
  Kennwort vorhanden sein.
- Ordnerfreigaben mit NTFS-Rechtematrix
- Ein PC als Dateiserver mit zweitem Datenträger, Netzlaufwerk per Anmeldeskript
- Zentral freigegebener Drucker, NAS zur Datensicherung

## Windows als Router
Ein PC mit zwei Netzwerkkarten verbindet das eigene Netz mit einem Nachbarnetz.

```powershell
Set-NetIPInterface -InterfaceAlias 'LAN-intern','LAN-extern' -Forwarding Enabled
route -p add 10.0.42.0 mask 255.255.255.0 10.0.50.2   # statische Route zum Nachbarnetz
route print
```

Dazu: ICMP-Echo in der Firewall erlauben, auf den Clients den Router als Standardgateway eintragen und auf der
Gegenseite die Rückroute setzen.

## Typische Probleme
| Problem | Ursache oder Lösung |
|---|---|
| PCs erscheinen nicht in der Netzwerkumgebung | Dienste der Funktionssuche prüfen, direkt per `\\PC-Name` testen |
| Zugriff verweigert | Konto fehlt auf dem Ziel-PC, anderes Kennwort oder NTFS strenger als die Freigabe |
| Kein Ping über den Router | Weiterleitung aus, ICMP blockiert, Gateway oder Rückroute fehlt |

## Was die Domäne verbessert
Benutzer, Rechte und Einstellungen werden zentral verwaltet, siehe [Active Directory](09-active-directory.md).
