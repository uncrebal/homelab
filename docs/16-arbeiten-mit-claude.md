# Arbeiten mit Claude

Viele Teile dieses Labors sind zusammen mit dem KI-Assistenten Claude entstanden. Hier halte ich fest, wie ich ihn
eingerichtet habe, wofür ich ihn nutze und wo ich bewusst Grenzen ziehe.

## Wie ich Claude nutze
- **Erklären:** Ich lasse mir Begriffe und Fehlermeldungen in einfachen Worten erklären, bevor ich etwas umsetze.
- **Fehlersuche:** Claude liest Logs und Konfigurationen und schlägt Ursachen vor. Ob es stimmt, prüfe ich am System.
- **Doku:** Aus dem, was wir gemacht haben, entstehen Anleitungen in meinem Wiki und hier auf GitHub.
- **Code:** Bei der Pokémon-App und kleineren Skripten hat Claude den Großteil geschrieben. Dort sage ich das auch
  offen dazu.

## Was eingerichtet ist

| Verbindung | Wofür | Rechte |
|---|---|---|
| Claude Desktop mit Claude Code | Befehle ausführen, Dateien lesen und schreiben auf meinem Laptop | Jede Aktion muss ich bestätigen oder vorher erlaubt haben |
| SSH-Schlüssel zum Proxmox-Host und zu den Linux-VMs | Zustand prüfen, Konfiguration auslesen, Änderungen nach Absprache | Schlüssel statt Passwort, ohne Passwort für sudo |
| BookStack-Connector | Anleitungen im Wiki anlegen und aktualisieren | eigener API-Token, nur für das Wiki |
| GitHub-Kommandozeile (`gh`) | Repositories anlegen und Änderungen hochladen | angemeldet mit meinem Konto |
| Browser (eingebaut bzw. Chrome-Erweiterung) | Webseiten und Dokus lesen, eigene Seiten prüfen | sichtbar, ich kann jederzeit eingreifen |
| Microsoft-Learn-Doku | offizielle Microsoft-Dokumentation zu Windows Server und PowerShell nachschlagen | nur lesen |
| Geplante Aufgaben | z. B. eine einmalige, rein lesende Prüfung, ob das Wochen-Backup gelaufen ist | nur lesen, ändert nichts |
| Gedächtnis (Memory) | Projektstand und meine Vorlieben zwischen Sitzungen behalten | lokale Textdateien |

Der BookStack-Connector braucht in BookStack ein API-Token (Profil → API-Tokens) und in der Claude-App die
Wiki-Adresse ohne `/` am Ende, sonst kommt HTTP 404.

## Regeln, die ich mir gesetzt habe
- Vor größeren Eingriffen gibt es einen **Snapshot oder ein Backup**.
- Claude arbeitet Schritt für Schritt und **nacheinander**, nicht parallel, weil die Festplatten sonst überlastet sind.
- Passwörter und Tokens tippe ich selbst ein. Sie stehen nie in der Doku und nicht im Chat.
- Bei veröffentlichten Inhalten werden echte Adressen, Namen und Zugangsdaten durch Beispielwerte ersetzt.
- Was Claude vorschlägt, will ich verstehen, bevor ich es übernehme. Wenn ich etwas nicht erklären kann, schreibe
  ich das dazu.

## Risiken, die mir bewusst sind
Ein KI-Assistent mit SSH-Zugang zum Server kann im schlimmsten Fall genauso viel kaputtmachen wie ich selbst. In
meinem Labor halte ich das für vertretbar, weil es keine produktiven Daten gibt und alles gesichert ist. In einer
Firma würde ich so einen Zugang nur mit eingeschränkten Rechten, Freigabe durch einen Menschen und Protokollierung
einrichten.

KI-Antworten können falsch sein. Deshalb steht in der Doku jeweils dabei, was getestet ist und was nicht.

## Weitere Projekte mit Claude
- **Pokémon-Sammlung:** Web-App mit PostgreSQL, siehe [eigenes Repository](https://github.com/uncrebal/pokemon-sammlung)
- **Lern-Wiki:** Unterrichtsmitschriften, Übungen und Anleitungen in BookStack, laufend ergänzt
- **Spieleprojekt in Godot:** ein Hobbyprojekt zum Ausprobieren, nicht öffentlich
