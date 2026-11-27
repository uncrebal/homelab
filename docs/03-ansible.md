# Ansible

Ansible verwaltet Linux-VMs, Container und den Proxmox-Host von einer VM aus. Auf den Zielen sind nur SSH und
Python nötig. Die Dateien liegen in [`../ansible`](../ansible/) (Auszug, mit Beispieladressen).

## Aufbau
- Inventory mit Gruppen (`proxmox`, `container`, `vms`, darunter `ubuntu`, `router`, `debian`)
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

## Das Wichtigste in Kürze
- Ein Playbook beschreibt, wie der Zustand sein soll, nicht jeden einzelnen Schritt.
- Man kann es beliebig oft laufen lassen. Ist schon alles richtig, ändert es nichts (idempotent).
- Mit `--check` sieht man vorher, was sich ändern würde.
- Proxmox selbst aktualisiere ich bewusst von Hand, nicht per Playbook.

Ansible ist für mich noch neu. Die Playbooks hier sind mit KI-Hilfe entstanden, ich habe sie im Labor ausprobiert.
