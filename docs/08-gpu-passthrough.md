# GPU-Passthrough

Ein kleines Nebenprojekt: Der Monitor am Server sollte als zusätzlicher Bildschirm für meinen Laptop dienen.

Dafür habe ich die Grafikkarte des Servers direkt an eine Debian-VM durchgereicht. Auf dem Laptop läuft
Sunshine und streamt einen virtuellen Monitor, die VM zeigt den Stream mit Moonlight auf dem Bildschirm am Server an.

Die Einrichtung auf dem Host war ziemlich technisch und ist größtenteils mit KI-Hilfe entstanden. Das Prinzip:
Proxmox darf die Grafikkarte nicht selbst benutzen, damit die VM sie bekommen kann. In der VM wird sie dann unter
*Hardware → PCI Device* hinzugefügt.

Es funktioniert, ist aber eher eine Spielerei als etwas, das man im Betrieb braucht.
