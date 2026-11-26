# Clients verwalten: Software und Einstellungen per GPO

Ziel: Neue Clients bekommen Programme und Vorgaben automatisch. Namen und Adressen sind Beispielwerte.

## Vorbereitung
- Neue Computer landen in `CN=Computers`. Darauf lässt sich keine GPO verknüpfen, deshalb in eine OU verschieben.
- **Admin-PC** mit **Windows Admin Center**, um den Server aus der Ferne zu verwalten:
  `winget install --id Microsoft.WindowsAdminCenter --exact`

## Software-Freigabe
Die Installation beim Systemstart läuft mit dem **Computerkonto**. Deshalb brauchen die *Domänencomputer*
Leserecht, nicht nur die Benutzer.

```powershell
New-SmbShare -Name Programme -Path E:\Programme -ReadAccess Jeder
icacls E:\Programme /grant 'AD\Domänencomputer:(OI)(CI)RX'
```

## Verteilung
*Computerkonfiguration → Richtlinien → Softwareeinstellungen → Softwareinstallation → Neu → Paket → Zugewiesen*

Die Quelle **muss ein Netzwerkpfad** sein (`\\DC1\Programme\...`), kein lokaler Pfad des Servers.

| Programm | Weg |
|---|---|
| Firefox, Chrome, ein Screenshot-Werkzeug | MSI über Softwareinstallation |
| LibreOffice | MSI nach Anpassung (siehe unten) |
| Scan-Werkzeug | Paket über Softwareinstallation |
| Evernote | nur als EXE: PowerShell-Startskript |

### LibreOffice: Paket lässt sich nicht hinzufügen
Die mehrsprachige MSI hat im Feld *Languages* sehr viele Sprach-IDs, damit kam die GPO-Softwareinstallation nicht
zurecht. In einer **Kopie** der MSI wurde die Liste gekürzt, danach lief die Verteilung. Das Original blieb
unverändert. Eine so geänderte Datei sollte man vorher auf ihre Herkunft prüfen (`Get-AuthenticodeSignature`
am Original).

### Startskript für eine EXE
Unter *Skripts → Start* im Reiter **PowerShell-Skripts** eintragen.

```powershell
Start-Transcript -Path C:\install-app.log -Force
$exe = Get-ChildItem '\\DC1\Programme' -Filter 'Evernote-latest*.exe' | Select-Object -First 1
$da  = Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*',
                        'HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*' -ErrorAction SilentlyContinue |
       Where-Object DisplayName -like 'Evernote*'
if ($exe -and -not $da) {
    $p = Start-Process $exe.FullName -ArgumentList '/AllUsers /S' -Wait -PassThru
    "Exitcode: $($p.ExitCode)"
}
Stop-Transcript
```

Das Skript installiert nur, wenn das Programm fehlt, und schreibt ein Protokoll.

## Browser-Vorgaben
1. **Zentralen Speicher** für ADMX-Vorlagen in SYSVOL anlegen und gleich die Windows-Vorlagen hineinkopieren,
   sonst fehlen diese in der Verwaltung.
2. ADMX-Vorlagen für Chrome und Firefox ergänzen.
3. Startseite und Verhalten beim Start setzen. Kontrolle in Firefox über `about:policies`.
4. Standardbrowser über eine **Standardzuordnungsdatei**: auf einem Musterclient einstellen, mit
   `Dism /Online /Export-DefaultAppAssociations:C:\apps.xml` exportieren und den Pfad per GPO vorgeben.

**Stand:** Die Firefox-Startseite funktioniert. Ob Firefox bei einem neu angemeldeten Benutzer wirklich
Standardbrowser ist, ist noch nicht getestet.

## Weitere Richtlinien
- **Windows Update** per GPO eingerichtet und geprüft
- **Automatisches Herunterfahren um 22 Uhr** über eine geplante Aufgabe (Konto SYSTEM, `shutdown.exe /s /f /t 60`)

## Härtung nach BSI
Die GPO-Einstellungen wurden mit den Windows-Härtungsempfehlungen des BSI (Studie SiSyPHuS) **verglichen und
dokumentiert, aber nicht angewendet**. Beispiel: Sperre bei Inaktivität nach höchstens 900 Sekunden. Die Studie
bezieht sich auf Windows 10.
