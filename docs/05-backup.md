# Backup

## Grundgedanke
Gesichert wird vor allem, was sich **nicht neu erzeugen** lässt: Konfiguration und Daten. Systeme, die aus Vorlage
und Ansible wieder entstehen, brauchen nicht unbedingt ein Abbild. Die von Hand gewachsene Docker-VM wird komplett
gesichert.

| Teil | Verfahren |
|---|---|
| Proxmox-Konfiguration | `tar` (ohne private Schlüssel) |
| MariaDB | `mysqldump` |
| Zabbix | `pg_dump` ohne Messverlauf, dazu die Konfiguration |
| Docker-VM | `vzdump` im Snapshot-Modus, zstd |
| OPNsense | `config.xml` nach Änderungen von Hand exportieren |

## Ablauf
Ein systemd-Timer startet wöchentlich ein Skript. Es sammelt die Sicherungen ein. Ein zweiter Schritt lädt sie mit
`rclone` in einen Cloud-Speicher, entfernt alte Stände und schreibt den Zeitpunkt des letzten Erfolgs in eine Datei.
Das Sammeln läuft mit normalen Rechten, nur der Upload als root. Die Einrichtung erfolgt per Ansible. Skripte und
Playbook dafür sind hier noch nicht enthalten.

## Einzelne VM von Hand sichern
Von einem Windows-Rechner aus. Der Umweg über `cmd /c` ist nötig, weil die Windows PowerShell 5.1 Binärdaten beim
Umleiten verändert:

```powershell
cmd /c "ssh root@10.0.10.10 vzdump 114 --mode snapshot --compress zstd --stdout > vzdump-qemu-114.vma.zst"
```

Wiederherstellen:

```bash
qmrestore /var/lib/vz/dump/vzdump-qemu-114.vma.zst 114 --storage local-lvm   # VM
pct restore 102 /var/lib/vz/dump/vzdump-lxc-102.tar.zst --storage local-lvm  # Container
```

Vorher auf dem Zielsystem die gleichen Bridges anlegen.

> Ein Backup, das nie zurückgespielt wurde, ist nur eine Hoffnung. Regelmäßige Wiederherstellungstests sind noch
> offen.
