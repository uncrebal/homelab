# Active Directory: Benutzer, Rechte, Laufwerke

Übungsprojekt für eine kleine Firma mit vier Abteilungen. Namen, Domäne und Adressen sind Beispielwerte.
Installation des Domänencontrollers: [Windows-Domäne](07-windows-domaene.md).

## Struktur

| Abteilung | Benutzer | Globale Gruppe |
|---|---|---|
| Geschäftsleitung | `gl01` | `G-GL` |
| Sekretariat | `sek01` | `G-SEK` |
| Vertrieb | `vt01` | `G-VT` |
| Entwicklung | `dev01` | `G-DEV` |

Dazu kommt ein eigenes Administratorkonto in den Domänen-Admins. Benutzer und Gruppen liegen in einer eigenen OU
`Firma`, die Computer in der Unter-OU `Clients`.

```powershell
$OU = 'OU=Firma,DC=ad,DC=example,DC=internal'
New-ADOrganizationalUnit -Name Firma -Path 'DC=ad,DC=example,DC=internal'
foreach ($g in 'GL','SEK','VT','DEV') { New-ADGroup -Name "G-$g" -GroupScope Global -Path $OU }

$pw = Read-Host 'Startkennwort' -AsSecureString
New-ADUser -Name vt01 -SamAccountName vt01 -Path $OU -AccountPassword $pw -Enabled $true -ChangePasswordAtLogon $true
Add-ADGroupMember -Identity G-VT -Members vt01
```

## NTFS-Rechte und Freigabe
**Prinzip:** Die Freigabe ist weit offen, die eigentliche Zugriffssteuerung machen die NTFS-Rechte.
`Ä` = Ändern, `L` = Lesen, `–` = kein Zugriff. Administratoren und SYSTEM haben überall Vollzugriff.

| Ordner | GL | SEK | VT | DEV |
|---|---|---|---|---|
| GL\Projekte | Ä | – | – | Ä |
| GL\Termine | Ä | Ä | – | – |
| SEK\Briefe | Ä | Ä | – | – |
| VT\Preise | Ä | L | Ä | – |
| DEV\Quellcode | Ä | – | – | Ä |
| DEV\Tools | L | – | – | Ä |

- Vererbung an der Wurzel trennen und die Gruppe *Benutzer* entfernen
- Auf den übergeordneten Ordnern nur *Lesen, Ausführen* für **diesen Ordner**, damit man hindurchgehen kann
- **Zugriffsbasierte Aufzählung**: Benutzer sehen nur Ordner, auf die sie Zugriff haben
- Rechte immer an Gruppen vergeben, nicht an einzelne Benutzer

```powershell
icacls E:\Daten\VT\Preise /inheritance:r
icacls E:\Daten\VT\Preise /grant '*S-1-5-32-544:(OI)(CI)F' '*S-1-5-18:(OI)(CI)F'
icacls E:\Daten\VT\Preise /grant 'AD\G-VT:(OI)(CI)M' 'AD\G-GL:(OI)(CI)M' 'AD\G-SEK:(OI)(CI)R'
New-SmbShare -Name Daten -Path E:\Daten -FullAccess Jeder
Set-SmbShare -Name Daten -FolderEnumerationMode AccessBased -Force
```

Die SIDs (`S-1-5-32-544` = Administratoren, `S-1-5-18` = SYSTEM) funktionieren unabhängig von der Systemsprache.

## Netzlaufwerke per Gruppenrichtlinie
*Benutzerkonfiguration → Einstellungen → Windows-Einstellungen → Laufwerkzuordnungen*, GPO mit der OU verknüpfen.
Die Richtlinie wirkt nur für Benutzer in dieser OU. Verbunden wird bei der Anmeldung.

## Home-Laufwerke
Persönliche Dateien liegen auf dem Server und damit in der Serversicherung.

```powershell
New-Item E:\Home -ItemType Directory
New-SmbShare -Name Home$ -Path E:\Home -FullAccess Jeder
```

Im Benutzerkonto unter *Profil → Basisordner* `H:` mit `\\DC1\Home$\%username%` verbinden.

**Stand:** Freigabe und Basisordner sind eingerichtet. Der Rechtetest (Benutzer A darf nicht in den Ordner von B)
ist noch nicht dokumentiert. Ordnerumleitung für Desktop und Dokumente ist nicht eingerichtet.

## Fehlersuche
| Werkzeug | Wofür |
|---|---|
| `gpresult /r /scope computer` bzw. `/scope user` | Welche Richtlinien kommen an? |
| `gpresult /h C:\gp.html` | Ausführlicher Bericht mit Komponentenstatus |
| *Effektiver Zugriff* in den erweiterten Sicherheitseinstellungen | Tatsächliche NTFS-Rechte eines Benutzers prüfen |
| `nslookup ad.example.internal` | Nutzt der Client den DC als DNS? Häufigste Ursache beim Domänenbeitritt |
| `ipconfig /all` | `169.254.x.x` heißt: keine Adresse vom DHCP |
