# Docker, Portainer und BookStack

Auf einer Ubuntu-VM laufen meine Docker-Dienste. Das wichtigste davon ist BookStack, mein Wiki: Dort stehen alle
Anleitungen zum Labor und meine Unterlagen aus der Umschulung. Diese GitHub-Doku ist eine bereinigte Auswahl daraus.

## Die Docker-VM
- Klon meiner Ubuntu-Server-Vorlage: 2 Kerne, 8 GB RAM, 60 GB Platte, im Labornetz hinter der OPNsense
- Startet automatisch mit dem Proxmox-Host, sonst wären Wiki und Monitoring nach einem Neustart weg
- Docker aus den Ubuntu-Paketquellen: `sudo apt install -y docker.io docker-compose-v2`

**Warum eine VM und kein Container?** Docker in einem LXC-Container geht zwar, braucht aber Sonderrechte.
In einer eigenen VM ist das sauberer getrennt.

## Portainer
Eine Weboberfläche für Docker. Damit starte und stoppe ich Container, schaue Logs an und lege **Stacks** an.
Ein Stack ist eine Compose-Datei mit mehreren Containern, die zusammengehören.

```bash
sudo docker run -d -p 9443:9443 --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
```

Danach im Browser `https://<docker-vm>:9443` öffnen und innerhalb von 5 Minuten den Admin anlegen.

| Stack | Inhalt |
|---|---|
| `bookstack` | BookStack und eine MariaDB-Datenbank |
| `monitoring` | Prometheus, Grafana und Exporter ([Compose-Datei](../compose/monitoring.yml)) |

## BookStack
BookStack ordnet Inhalte wie eine Bibliothek: **Regale → Bücher → Kapitel → Seiten**. Ich habe z. B. ein Buch für
das Labor und je ein Buch pro Schulfach.

Der Stack besteht aus zwei Containern:

```yaml
services:
  bookstack:
    image: lscr.io/linuxserver/bookstack
    environment:
      APP_URL: http://<docker-vm>:6875
      APP_KEY: ${APP_KEY}
      DB_HOST: bookstack-db
      DB_USERNAME: bookstack
      DB_PASSWORD: ${DB_PASSWORD}
      DB_DATABASE: bookstackapp
    volumes: ["bookstack_data:/config"]
    ports: ["6875:80"]
    depends_on: [bookstack-db]
    restart: unless-stopped

  bookstack-db:
    image: lscr.io/linuxserver/mariadb
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: bookstackapp
      MYSQL_USER: bookstack
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes: ["bookstack_db:/config"]
    restart: unless-stopped

volumes:
  bookstack_data:
  bookstack_db:
```

Die Passwörter stehen nicht in der Datei, sondern werden in Portainer als Umgebungsvariablen gesetzt. Nach dem
ersten Login sofort E-Mail und Passwort des Standard-Admins ändern.

### Stolperfallen
- `APP_URL` muss genau stimmen, sonst kommt „Host is malformed“.
- Das Datenbank-Passwort muss bei beiden Containern gleich sein.
- Geänderte Einstellungen greifen erst, wenn der Container neu erstellt wird. Ein Neustart reicht nicht.
- Der PDF-Export großer Bücher brach mit Fehler 500 ab. Im Log stand, dass PHP zu wenig Speicher hatte. Lösung:
  Speicherlimit in der PHP-Konfiguration des Containers auf 1 GB erhöht.

## Backup des Wikis
| Was | Wie oft | Wohin |
|---|---|---|
| Datenbank-Dump und Dateien (Bilder, Anhänge) | täglich, 14 Tage aufbewahrt | Cloud-Speicher per `rclone` |
| Alle Bücher als PDF über die BookStack-API | wöchentlich | Cloud-Speicher, lesbar auch ohne BookStack |

Das Backup-Skript liest die Zugangsdaten aus den Umgebungsvariablen des Datenbank-Containers, im Skript selbst
steht kein Passwort. Für den PDF-Export gibt es ein eigenes API-Token.

**Wiederherstellung getestet:** Ich habe den Dump in einen leeren Test-Container eingespielt und die Zahl der
Bücher, Kapitel und Seiten mit dem Original verglichen. Beide waren identisch. Danach wurde der Test-Container
wieder gelöscht.

## Claude und BookStack
Claude ist über die BookStack-API angebunden und kann Seiten lesen und anlegen. Viele Anleitungen sind so
entstanden. Mehr dazu unter [Arbeiten mit Claude](16-arbeiten-mit-claude.md).
