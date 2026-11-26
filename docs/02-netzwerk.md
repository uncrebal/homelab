# Netzwerk und Zugriff

## OPNsense
Eine OPNsense-VM trennt das Labornetz vom Außennetz und übernimmt NAT, DHCP und DNS (Dnsmasq).
So brauchen die VMs dahinter keine Adresse im Außennetz.

| Schnittstelle | Netz |
|---|---|
| WAN (`vmbr0`) | `10.0.10.104/24` |
| LAN (`vmbr2`) | `10.0.20.1/24`, DHCP mit statischen Zuordnungen |
| Analyse-Lab (`vmbr6`) | `10.0.60.254/24` |

**Zugriff vom Verwaltungsrechner:** Firewall-Regeln erlauben dessen Adresse den Zugriff auf die Oberfläche und ins
LAN. Dazu kommt auf dem Rechner eine Route:

```powershell
route -p add 10.0.20.0 mask 255.255.255.0 10.0.10.104
```

Für die Ersteinrichtung vom WAN aus wurde der Paketfilter kurz mit `pfctl -d` abgeschaltet und nach dem Anlegen der
Regeln mit `pfctl -e` wieder eingeschaltet. Das schaltet den ganzen Schutz ab und ist nur in einem kontrollierten
Aufbau vertretbar.

## NetBird (VPN von außen)
Ein unprivilegierter LXC-Container dient als Routing-Peer. Auf dem Proxmox-Host selbst wird dafür kein
VPN-Client installiert.

Der Container braucht Zugriff auf `/dev/net/tun`, in `/etc/pve/lxc/<ID>.conf`:

```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Im NetBird-Dashboard werden nur die benötigten Ziele als Ressourcen angelegt und per Richtlinie auf die nötigen
Ports beschränkt (Weboberfläche, SSH, bei Bedarf RDP).

**Stolperfalle:** Nutzt das Netz, aus dem man sich verbindet, denselben IP-Bereich wie das Labor, gehen die Pakete
ins lokale Netz statt in den Tunnel.

## fail2ban auf dem Host
Sperrt Quelladressen nach mehreren falschen Anmeldungen, für SSH und für die Proxmox-Weboberfläche.

`/etc/fail2ban/jail.local`:

```ini
[DEFAULT]
ignoreip = 127.0.0.1/8 100.64.0.0/10
maxretry = 5
bantime = 10m

[proxmox]
enabled = true
port = https,http,8006
filter = proxmox
backend = systemd
findtime = 10m
```

`/etc/fail2ban/filter.d/proxmox.conf`:

```ini
[Definition]
failregex = pvedaemon\[.*authentication (failure|error); rhost=<HOST> user=.* msg=.*
ignoreregex =
```

`100.64.0.0/10` ist der Adressbereich des VPN. Die Ausnahme verhindert, dass man sich selbst aussperrt, bedeutet aber
auch, dass Anmeldeversuche aus diesem Bereich nicht gesperrt werden. Prüfen mit `fail2ban-client status proxmox`.
