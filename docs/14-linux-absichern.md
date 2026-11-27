# Debian absichern: sudo, SSH-Schlüssel, UFW

Durchgeführt auf einer Debian-VM, bedient per SSH.

## sudo statt root
1. `sudo` installieren, Benutzer aufnehmen: `usermod -aG sudo <benutzer>`
2. Neu anmelden und `sudo` testen. Dabei eine root-Sitzung offen lassen.
3. Erst dann das root-Kennwort sperren: `passwd -l root`

## SSH nur mit Schlüssel
| Schritt | Wo |
|---|---|
| `ssh-keygen -t ed25519` (mit Passphrase) | eigener Rechner |
| Öffentlichen Schlüssel in `~/.ssh/authorized_keys` eintragen | Server, als der Zielbenutzer |
| In `sshd_config` `PubkeyAuthentication yes`, `PasswordAuthentication no` | Server |
| `sudo systemctl restart ssh` | Server |

Test, ob das Kennwort wirklich abgeschaltet ist. Erwartet wird `Permission denied (publickey)`:

```bash
ssh -o PubkeyAuthentication=no <benutzer>@<server>
```

Während der Umstellung immer eine Sitzung offen lassen. Der private Schlüssel bleibt auf dem eigenen Rechner.

## Firewall mit UFW
```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh        # vor dem Einschalten, sonst sperrt man sich aus
sudo ufw enable
sudo ufw status verbose
```

UFW ist zustandsbehaftet: Antworten auf eigene Verbindungen kommen durch.

Ping blockieren geht über die Zeile für `echo-request` in `/etc/ufw/before.rules` (Kette `ufw-before-input`,
`ACCEPT` zu `DROP`). In der Praxis bringt das wenig Schutz und erschwert die Fehlersuche.

## Feste Adresse oder DHCP-Reservierung
Hier wurde eine Reservierung in OPNsense verwendet: zentral verwaltet, immer dieselbe Adresse. Nachteil: Ohne
DHCP-Server bekommt das System keine Adresse.
