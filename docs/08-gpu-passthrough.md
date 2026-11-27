# GPU-Passthrough

Ziel: Der Monitor am Server soll als zusätzlicher Bildschirm des Laptops dienen. Dafür bekommt eine Debian-VM
die Grafikkarte und zeigt mit Moonlight einen Stream, den Sunshine auf dem Laptop von einem virtuellen
Monitor sendet.

## Auf dem Host
```bash
dmesg | grep -i -E "DMAR|IOMMU"      # ist IOMMU aktiv?
lspci -nn | grep -i nvidia           # Geräte-IDs ermitteln
```

Vor dem Durchreichen die IOMMU-Gruppe prüfen. Hier lagen Grafik- und Audio-Teil der Karte allein in einer Gruppe.

```bash
# IDs durch die eigenen ersetzen
echo 'options vfio-pci ids=10de:13f0,10de:0fbb' > /etc/modprobe.d/vfio.conf
printf 'blacklist nouveau\nblacklist nvidia\nblacklist nvidiafb\n' > /etc/modprobe.d/blacklist.conf
printf 'vfio\nvfio_iommu_type1\nvfio_pci\n' >> /etc/modules
printf 'softdep snd_hda_intel pre: vfio-pci\nsoftdep nouveau pre: vfio-pci\n' > /etc/modprobe.d/vfio-softdep.conf
update-initramfs -u -k all
reboot
```

Kontrolle mit `lspci -nnk -s <Adresse>`: bei beiden Geräten steht `Kernel driver in use: vfio-pci`.

## VM
Maschinentyp q35, UEFI. Unter *Hardware → PCI Device* die Karte mit *All Functions* und *PCI-Express* hinzufügen.

Eine Schleife in der VM startet den Stream neu, wenn die Verbindung abbricht.
