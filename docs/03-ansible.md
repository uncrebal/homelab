# Ansible

Ansible verwaltet Linux-VMs, Container und den Proxmox-Host von einer VM aus. Auf den Zielen sind nur SSH und
Python nötig. Die Dateien liegen in [`../ansible`](../ansible/) (Auszug, mit Beispieladressen).

## Aufbau
- **Inventory** mit Gruppen (`proxmox`, `container`, `vms`, darunter `ubuntu`, `router`, `debian`)
- Unterschiede zwischen Systemen stehen in `group_vars` und `host_vars`, nicht in den Playbooks
- `become` (sudo) steht nur im Playbook, sonst fragt schon ein Verbindungstest nach dem Passwort

## Playbooks
| Datei | Aufgabe |
|---|---|
| `baseline.yml` | apt-Proxy, Updates, Grundpakete, Hinweis auf nötigen Neustart |
| `guest-agent.yml` | QEMU-Guest-Agent in Proxmox aktivieren und in den VMs installieren |

```bash
cd ansible
ansible-inventory --graph
ansible all -m ping
ansible-playbook playbooks/baseline.yml --check --diff -K   # Probelauf
ansible-playbook playbooks/baseline.yml -K
```

`ping` ist hier kein ICMP-Ping: Ansible meldet sich per SSH an und startet Python.

## Verwendete Techniken
| Technik | Wofür |
|---|---|
| `delegate_to` | Der Task läuft je VM, wird aber auf dem Proxmox-Host ausgeführt (dort gibt es `qm`) |
| `changed_when: false`, `check_mode: false` | Reines Lesen zählt nicht als Änderung und läuft auch im Probelauf |
| Bedingung vor `qm set` | **Idempotenz:** Ein zweiter Lauf setzt die Option nicht erneut |

`StrictHostKeyChecking=accept-new` in der `ansible.cfg` übernimmt unbekannte Host-Schlüssel beim ersten Kontakt.
Geänderte Schlüssel werden weiterhin abgelehnt.

Proxmox selbst wird bewusst nicht automatisch aktualisiert.
