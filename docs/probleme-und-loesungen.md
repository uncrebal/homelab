# Probleme und Lösungen

Hier sammle ich die Fehler, die mich beim Aufbau am meisten Zeit gekostet haben. Wo ich mir bei der Ursache nicht sicher bin, steht das dabei.

## 1. DHCP: Der Client bekommt keine Adresse
Symptom: In Wireshark erscheint nur *DHCP Discover*, nie ein *Offer*.

Suche:
1. Schnittstelle in OPNsense aktiv? War in Ordnung.
2. Firewall-Log: DHCP-Pakete wurden per *Default deny* blockiert. Eine Regel mit der Quelle „Netz der
   Schnittstelle“ reicht nicht, weil der Client noch von `0.0.0.0` an `255.255.255.255` sendet. Es braucht eine
   eigene Regel für UDP 68 → 67.
3. Danach keine Blockierung mehr, aber immer noch kein Offer.

Ursache: Dnsmasq lauschte laut Einstellung nur auf `LAN`. Nach dem Ergänzen der neuen Schnittstelle ging es.

Am Ende waren es zwei Fehler hintereinander. Der Mitschnitt zeigt, an welcher Stelle der Ablauf abbricht.
Das Firewall-Log von OPNsense zeigte die Zeit in UTC.

## 2. LVM-Thin-Pool wird voll, obwohl in der VM Platz frei ist
Symptom: Eine VM belegte ihre vollen 60 GB auf der SSD, nutzte innen aber nur 15 GB.

Ursache: Der Pool erfährt vom Löschen nur durch TRIM/discard. Die virtuelle Platte hatte kein `discard=on`.
Zusätzlich hatte der Ubuntu-Installer nur einen Teil der LVM-Gruppe belegt, den Rest erreicht `fstrim` nicht.

Lösung: `discard=on` setzen, VM kalt starten, `fstrim -av` ausführen und den ungenutzten Bereich mit einem
temporären Volume freigeben. Danach wurde discard per Ansible für weitere Linux-VMs eingeschaltet.

| | vorher | nachher |
|---|---|---|
| Platte der VM | 100 % | 25,7 % |
| Pool auf der SSD | 78,7 % | 37,5 % |

Auch das Backup-Volume ließ sich trimmen: Der betroffene Pool sank von 72,5 % auf 57,3 %.

## 3. Zabbix meldet „API service not available“
Ursache: Eine einzelne Prüfung des Templates verlangte ein vollständig gültiges Zertifikat. Proxmox nutzt hier
ein selbstsigniertes, die Verbindung brach schon beim Aufbau ab. Deshalb stand auch nichts im Proxmox-Log.

Lösung: `{$HTTP.TLS.VERIFY:"Proxmox VE"}` auf `none`. Die Verbindung bleibt verschlüsselt, aber die Identität
der Gegenseite wird nicht geprüft. Für mein Labor reicht das, sauberer wäre ein eigenes Zertifikat.

## 4. Ansible scheitert an der sudo-Abfrage auf neuem Ubuntu
Ursache: Ubuntu ab 25.10 nutzt `sudo-rs`. Dessen Passwort-Abfrage hat Ansible nicht erkannt.
Lösung: `ansible_become_exe: sudo.ws` für die Ubuntu-Gruppe.

## 5. apt hinter dem Paket-Cache: HTTPS-Quellen schlagen fehl
Ursache: apt nutzte den HTTP-Proxy auch für HTTPS, apt-cacher-ng lehnte den Tunnel ab.
Lösung: `Acquire::https::Proxy "DIRECT";` auf den Clients oder einzelne Quellen über `PassThroughPattern`
erlauben.

## 6. Guest-Agent installiert, aber ohne Funktion
Ursache: Die virtio-serial-Schnittstelle entsteht erst nach einem Kaltstart. Ein Neustart in der VM
reicht nicht.

## 7. GPU-Passthrough: Der Host belegt den Audio-Teil der Karte
Lösung: Beide Geräte-IDs an `vfio-pci` binden, die Module früh laden und mit `softdep` sicherstellen, dass
`vfio-pci` vor dem Audio-Treiber kommt.

## 8. Weboberfläche hängt beim Klonen
Symptom: Zwei parallele Klone auf dieselbe Festplatte, die Weboberfläche reagierte nicht mehr.
Lösung: Hängenden `lvcreate`-Prozess beenden, `pvestatd` neu starten, verwaiste Volumes prüfen und entfernen.
Seitdem wird nacheinander geklont.

## 9. Windows-VMs nachträglich auf VirtIO umstellen
Bei vorhandenen Windows-VMs wurde der Treiber offline mit `virt-v2v-in-place` eingespielt, danach wurden Platte
und Netzwerkkarte auf VirtIO umgestellt (gleiche MAC-Adresse).

- Bei ausstehenden Windows-Updates meldete der Linux-Treiber `ntfs3` Dateisystemfehler. Das Vorgehen: Updates
  fertig installieren lassen, für den Lauf das Einbinden erzwingen, danach in Windows `chkdsk C: /f`.
- Bei einem mit Sysprep generalisierten Image brach der eingefügte Firstboot-Dienst das Setup ab. Er wurde entfernt
  und durch `SetupComplete.cmd` ersetzt. Ob ein Klon dieser Vorlage damit sauber durchläuft, ist noch nicht
  bestätigt.

Vor solchen Eingriffen: Backup und Snapshot.

## 10. Windows: Nach einem Neustart bekommt kein Client eine Adresse
Suche: Mit fester Adresse ließ sich der Server anpingen, die Verbindung war also in Ordnung. Das DHCP-Audit-Log
meldete eine nicht erreichbare Domäne, eine fehlgeschlagene Autorisierung und das Ende des Dienstes.

Ursache: Der DHCP-Dienst war gestartet, bevor Active Directory bereit war.

Lösung: Dienst neu starten und auf verzögerten Start stellen:
`sc.exe config dhcpserver start= delayed-auto`

## 11. Softwareverteilung: „Installationsquelle nicht verfügbar“
Ursache: Beim Anlegen des Pakets wurde die Datei über das lokale Laufwerk des Servers ausgewählt. Der Client
suchte den Pfad auf seiner eigenen Platte.
Lösung: Paket entfernen und mit Netzwerkpfad (`\\DC1\Programme\...`) neu anlegen.

## 12. Startskript läuft nicht oder die Clients hängen
- Läuft nicht: Die `.ps1` stand im Reiter *Skripts* statt *PowerShell-Skripts*. Hinweis darauf war die fehlende
  Protokolldatei.
- Hängt: Eine frühere Version lud beim Start ein Windows-Feature nach. Über die langsame Leitung blieben die
  Clients über 20 Minuten bei „Bitte warten“, weil das Startskript vor der Anmeldung lief. Der unnötige Download
  wurde entfernt.

## 13. Computer lässt sich nicht in die OU verschieben
Ursache: In der OU gab es Benutzerkonten mit denselben Namen wie die Computer (Fehler 8305).
Lösung: Eine eigene Unter-OU `Clients`. Die Richtlinien der übergeordneten OU werden vererbt.
