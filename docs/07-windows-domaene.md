# Windows-Domäne

## Domänencontroller (Windows Server 2025)
- UEFI und TPM 2.0, VirtIO SCSI, Systemplatte auf der SSD, zweite Platte für Daten
- Zwei Netzwerkkarten: Internet (über OPNsense) und Domänennetz (statische IP `10.0.30.1`)
- Bei der Installation den `vioscsi`-Treiber von der virtio-win-CD laden, sonst wird keine Platte angezeigt

In der Oberfläche: *Server-Manager → Lokaler Server* → Computername ändern, dann *Verwalten → Rollen und Features
hinzufügen → Active Directory-Domänendienste*. Danach über die gelbe Flagge *Server zum Domänencontroller
heraufstufen → Neue Gesamtstruktur*. Dasselbe als PowerShell:

```powershell
Rename-Computer -NewName DC1 -Restart
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName ad.example.internal -InstallDns
```

DHCP verteilt nur im Domänennetz. In der Oberfläche: Rolle *DHCP-Server* hinzufügen, über die gelbe Flagge die
Konfiguration abschließen (autorisiert den Server im AD), dann in der DHCP-Konsole einen neuen Bereich mit den
Optionen Router und DNS anlegen und unter *Bindungen* nur die Domänenkarte anhaken. Als PowerShell:

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC
Add-DhcpServerv4Scope -Name Domaene -StartRange 10.0.30.11 -EndRange 10.0.30.50 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -DnsServer 10.0.30.1 -DnsDomain ad.example.internal
```

Zwei Netzwerkkarten an einem Domänencontroller sind für produktive Umgebungen nicht empfohlen. Hier war es eine
Vorgabe der Übung.

## Windows-11-Vorlage
1. Windows installieren, alle Updates einspielen
2. Geräteverschlüsselung (BitLocker) ausschalten und die Entschlüsselung abwarten
3. Verallgemeinern: `C:\Windows\System32\Sysprep\sysprep.exe /generalize /oobe /shutdown`
4. Nicht erneut starten, in Proxmox in ein Template umwandeln
5. Clients als *Full Clone* erzeugen, ins Domänennetz hängen und beitreten. In der Oberfläche über
   *Einstellungen → System → Info → Domäne oder Arbeitsgruppe*, oder:

```powershell
Add-Computer -DomainName ad.example.internal -Restart
```

Wenn Sysprep abbricht: Das Log liegt in `C:\Windows\System32\Sysprep\Panther\setupact.log`. Ursachen waren
ausstehende Updates und einzelne AppX-Pakete.

Hinweis: Ist der Domänencontroller aus und bei einem Rechner als einziger DNS-Server eingetragen, funktioniert
dort die Namensauflösung nicht.
