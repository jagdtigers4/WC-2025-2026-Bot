# WC-2025-2026-Bot
The Settlers IV World Championship 2025/2026 Bot
# User Manual – mainbotv2

## 1. Zweck des Bots
Dieser Bot verwaltet ein komplettes Turnier auf Discord. Der Schwerpunkt liegt auf Match-Organisation, Terminabstimmung, Match-Setup, Ergebnismeldung, KO-Stage, Tippspiel, Bonusfragen, Teilnehmerverwaltung, Streamer-Slots und HTML-Exporten für externe Webseiten.

Der Bot ist in diesem Paket **absichtlich bereinigt**:
- alle Discord-IDs wurden durch gut erkennbare Platzhalter ersetzt
- der Bot-Token wurde durch einen Platzhalter ersetzt

Bevor der Bot gestartet werden kann, müssen diese Platzhalter wieder durch echte Werte ersetzt werden.

## 2. Voraussetzungen
- Python 3.11 oder neuer empfohlen
- Installierte Pakete aus `requirements.txt`
- Discord-Bot mit passenden Rechten auf dem Zielserver
- Ein korrekt ausgefülltes `config.json`

Installation:
```bash
pip install -r requirements.txt
```

Start:
```bash
python bot.py
```

## 3. Wichtige Platzhalter im Paket
Im gesamten Paket stehen jetzt Platzhalter wie:
- `<DISCORD_BOT_TOKEN>`
- `<DISCORD_GUILD_ID>`
- `<DISCORD_CHANNEL_ID_001>`
- `<DISCORD_USER_ID_001>`
- `<DISCORD_MESSAGE_ID_001>`

Diese Platzhalter müssen vor dem produktiven Einsatz ersetzt werden.

## 4. Zentrale Dateien
### `config.json`
Hauptkonfiguration des Bots.
Wichtige Einträge:
- `bot_token`: Bot-Token
- `guild_id`: Server-ID
- `roles`: Namen der verwendeten Discord-Rollen
- `turnierleitung_ids`: Discord-IDs mit erweiterten Rechten
- verschiedene `..._channel_id`-Felder für Orga, Ergebnisse, Termine, Hilfe, Cancel, Reminder, Teilnehmer etc.
- `applications_source`: Quelle für Bewerbungen/Anmeldungen
- `participations.source`: Quelle für Teilnehmerliste
- `hostinger_termine` / `hostinger_results`: Exportoptionen für Webseiten-Snippets
- `swiss`: Swiss-Stage-Konfiguration

### Laufzeit-/Statusdateien
Diese Dateien speichert der Bot selbst:
- `matches.json`: Matchstatus, Pick-Status, Fortschritt, Meta-Daten
- `termine.json`: bestätigte Termine
- `unconfirmed.json`: offene Terminvorschläge
- `cancel_requests.json`: offene Stornoanfragen
- `winner_pending.json`: offene Ergebnisbestätigungen
- `players.json`: bekannte Spieler und Zuordnung Discord-ID → Ingame-Name
- `ko_bracket.json`: KO-Baum und aktueller Stage-Status
- `played_maps_ko.json`: bereits verwendete KO-Maps
- `tippspiel.json`: Tippspiel-Daten
- `bonus_tips.json` / `bonus_tips1.json`: Bonusfragen-Daten
- `streamers.json`: Streamer-Profile und Zuordnungen
- `applications_state.json` / `apps_state.json`: Bewerbungs-/Applications-Zustand
- `civ_stats.json`: Völkerstatistik
- `ergebnisse.json`: Ergebnisse
- `reminders.json` / `stream_reminders.json`: Reminder-Daten

## 5. Rollen und Berechtigungen
Der Bot verwendet Rollen aus `config.json`, z. B.:
- Owner
- Admin
- Streamer
- Player
- Viewer

Viele Slash-Commands sind nur für Orga/Admin/Owner gedacht. Der Bot prüft das an mehreren Stellen.

## 6. Match-Workflow im Überblick
### 6.1 Match-Channel
Jeder Match-Channel besitzt einen persistenten Start-View mit typischen Aktionen:
- Hilfe anfordern
- Termin vorschlagen
- Ergebnis melden

### 6.2 Terminabwicklung
Ablauf:
1. Ein Spieler schlägt einen Termin vor.
2. Der Gegner bestätigt oder lehnt ab.
3. Bei Bestätigung wird der Termin gespeichert.
4. Danach kann bei Bedarf eine Cancel-Anfrage gestellt werden.
5. Eine Stornierung muss erst von der Orga bestätigt werden.

Wichtige Eigenschaften:
- kein Spam durch parallele Vorschläge
- bestätigte Termine werden separat gespeichert
- offene Vorschläge und Cancel-Requests blockieren widersprüchliche Aktionen

### 6.3 Check-In
Mit `/checkin` bestätigen Spieler ihre Anwesenheit für ein Match. Der Bot speichert dabei den Ingame-Namen für das konkrete Match.

### 6.4 Matchstart
Mit `/matchstart` startet der eigentliche Game-Setup-Flow.

Swiss/Standard-Flow:
- Münzwurf
- Moduswahl
- je nach Modus Bans / Picks / Positionswahl
- Map-Ausgabe

KO-Flow:
- Münzwurf
- Counterpick-/Blindpick-/Normalpick-Logik je nach Spielsituation
- Positionswahl
- Map-Datei

Grand Final:
- bei Game 3 können die Previews für Game 3 und Game 4 zusammen erscheinen
- der eigentliche Ablauf pro Game bleibt trotzdem getrennt

### 6.5 Ergebnis melden
Spieler können ein Ergebnis melden.
Danach muss der Gegner bestätigen oder ablehnen.
Erst nach der Bestätigung wird das Ergebnis dauerhaft übernommen.

## 7. Map- und Preview-System
Der Bot nutzt Map-Dateien und Preview-Bilder aus lokalen Ordnern.
Wichtige Logik aus dem Code:
- Maps werden aus dem `Maps`-Bereich gelesen
- Previews werden aus `MapPreviews` bzw. verwandten Verzeichnissen geladen
- bereits gespielte KO-Maps werden in `played_maps_ko.json` getrackt
- gespielte Maps/Previews können automatisch in `GespielteMaps` und `GespielteMapPreviews` verschoben werden
- Mapper sollen nach Möglichkeit nicht ihre eigenen Maps bekommen

## 8. KO-Stage
### Wichtige Commands
- `/ko_import` – importiert die Start-Matchups der KO-Phase
- `/gen_kostage` – erstellt Matchchannels für die aktuelle KO-Runde
- `/gen_next` – erzeugt die nächste KO-Runde, wenn die aktuelle vollständig ist
- `/ko_testfill` – Testfunktion zum zufälligen Füllen von Gewinnern/Scores

### KO-Game-Logik
Der Bot unterstützt mehrere Games in einer Serie und speichert, welches Game als Nächstes gespielt wird. Je nach Stand werden Counterpick, Blindpick oder andere Views eingeblendet.

## 9. Tippspiel
### Zweck
Separates Prediction-Game für Zuschauer/Teilnehmer.

### Wichtige Commands
- `/tipps_setup` – initialisiert das Tippspiel und postet den Einstieg
- `/tipps` – Tipps abgeben oder ändern
- `/tipps_leaderboard` – Rangliste anzeigen
- `/tipps_view` – Tipps eines bestimmten Users ansehen
- `/tipps_selfcheck` – eigene Tipps prüfen
- `/tipps_ban` – User vom Tippspiel ausschließen
- `/tipps_unban` – Sperre wieder entfernen
- `/tipps_points` – Punkte manuell korrigieren
- `/tipps_recalc` – komplette Neuberechnung aller Tippspiel-Punkte
- `/tipps_tendenzen` – Tendenzen für KO-Tipps posten

### Typische Funktion
- Check-in für Teilnahme
- Abgabe/Änderung von Tipps
- permanentes Leaderboard
- manuelle Korrektur durch Orga möglich

## 10. Bonusfragen
Zusätzlich zum normalen Tippspiel gibt es Bonusfragen.

Wichtige Commands:
- `/bonus_setup` – Bonus-Overlay posten oder aktualisieren
- `/bonus_tip` – Antwort auf Bonusfrage abgeben
- `/bonus_reset` – Bonus-Tipps zurücksetzen
- `/bonus_publish` – Bonus-Tendenzen öffentlich posten

## 11. Teilnehmerverwaltung / Applications
Der Bot kann Bewerbungen bzw. Registrierungen aus einer externen Quelle einlesen.

Wichtige Commands:
- `/apps_refresh` – Applications-Panel manuell aktualisieren
- `/test_registration_dm` – Registrierungs-DM testweise an sich selbst schicken
- `/assign_player_roles` – bekannten Teilnehmern automatisch die Player-Rolle geben
- `/post_participants` – Teilnehmerliste in den konfigurierten Channel posten

Zusatzfunktionen:
- Discord-ID setzen
- Bewerbung annehmen/ablehnen
- bei Annahme Rolle vergeben und Nickname setzen

## 12. Streamer-Funktionen
Der Bot unterstützt Streamer-Profile und Slot-Handling.

Wichtige Commands:
- `/streamer_profile` – Streamer-Profil setzen
- `/link` – VOD/Stream-Link einem Match zuordnen

Im Code sind außerdem Stream-Slot-Views enthalten:
- begrenzte Slot-Anzahl
- Abmeldung möglich
- Zuordnung über Discord-ID

## 13. Export-Funktionen
Für externe Webseiten oder HTML-Snippets gibt es mehrere Exporte.

Commands:
- `/export_participants_html`
- `/export_participants`
- `/export_termine_html`
- `/export_ergebnisse_html`
- `/export_participants_items_de`
- `/export_participants_items_en`
- `/export_termine_items_de`
- `/export_termine_items_en`

Zusätzlich existieren in `config.json` Optionen für Hostinger-/HTML-Ausgaben.

## 14. Admin- und Wartungs-Commands
- `/sync_commands` – Slash-Commands neu synchronisieren
- `/roles_setup` – Rollen und Channel-Berechtigungen neu einrichten
- `/bot_status` – Presence ändern
- `/bot_avatar` – Bot-Avatar setzen
- `/bot_nick` – Bot-Nickname auf dem Server setzen
- `/gen_matches` – Match-Channels erzeugen
- `/del_matches` – Match-Kategorien löschen
- `/clear_results` – Ergebnisse aufräumen
- `/clear_termine` – bestätigte Termine löschen
- `/clear_all` – kompletter Reset für Termine und Ergebnisse
- `/matches_overview_refresh` – Match-Übersicht neu aufbauen

## 15. Force- und Notfall-Commands
Für Sonderfälle gibt es Befehle, die direkt in den Turnierstand eingreifen.

Wichtige Commands:
- `/force_setwinner` – Sieger direkt setzen
- `/force_reset_channel` – Match-Channel auf Anfangszustand zurücksetzen
- `/forcewinner` – einzelnen Game-Sieg mit Völkern/Stati setzen
- `/add_player_to_match` – Spieler nachträglich in einen Match-Channel aufnehmen
- `/stati` – Statistikbild an ein Ergebnis anhängen

Diese Commands sollten nur von der Orga genutzt werden.

## 16. Typische Erstkonfiguration
1. `config.json` öffnen.
2. Alle Platzhalter für Guild, Channels, User-IDs und Token ersetzen.
3. Rollen auf dem Discord-Server anlegen oder vorhandene Rollennamen korrekt eintragen.
4. Optional externe Quellen für Teilnehmer und Bewerbungen eintragen.
5. Bot mit den nötigen Rechten auf den Server einladen.
6. `pip install -r requirements.txt`
7. `python bot.py`
8. Danach `/sync_commands` und ggf. `/roles_setup` ausführen.

## 17. Typischer Betrieb
### Swiss Stage
1. Teilnehmerdaten prüfen
2. Match-Channels generieren
3. Spieler machen Terminabstimmung
4. `/checkin`
5. `/matchstart`
6. Ergebnis melden und bestätigen lassen
7. Exporte / Übersichten aktualisieren

### KO Stage
1. `/ko_import`
2. `/gen_kostage`
3. Pro Match normaler Ablauf mit `/matchstart`
4. Nach Abschluss einer Runde `/gen_next`

## 18. Wichtige Hinweise
- Viele Views sind persistent. Nach einem Bot-Neustart können laufende UIs weiterverwendet werden, solange die zugrunde liegenden Daten noch vorhanden sind.
- Der Bot speichert viel Status in JSON-Dateien. Vor größeren Änderungen sind Backups sinnvoll.
- Mehrere Commands greifen direkt in den Turnierzustand ein. Diese nur mit klaren Rollenbeschränkungen verwenden.
- Die aktuelle Paketversion ist eine bereinigte Vorlage. Ohne Ersetzen der Platzhalter ist sie nicht produktiv lauffähig.

## 19. Schnelle Fehlerprüfung
Wenn etwas nicht funktioniert, zuerst prüfen:
- sind alle Platzhalter ersetzt?
- stimmt `guild_id`?
- stimmen die Channel-IDs?
- sind die Rollennamen in `config.json` korrekt?
- hat der Bot ausreichende Discord-Rechte?
- existieren benötigte JSON-Dateien und Map-/Preview-Ordner?

## 20. Kurzfassung der wichtigsten Slash-Commands
### Match / Turnier
- `/checkin`
- `/matchstart`
- `/gen_matches`
- `/gen_kostage`
- `/gen_next`
- `/ko_import`
- `/forcewinner`
- `/force_setwinner`

### Tippspiel / Bonus
- `/tipps`
- `/tipps_leaderboard`
- `/tipps_setup`
- `/tipps_recalc`
- `/bonus_setup`
- `/bonus_tip`
- `/bonus_publish`

### Verwaltung
- `/apps_refresh`
- `/post_participants`
- `/assign_player_roles`
- `/roles_setup`
- `/sync_commands`
- `/matches_overview_refresh`

### Exporte
- `/export_participants_html`
- `/export_termine_html`
- `/export_ergebnisse_html`

---
Diese Anleitung beschreibt die im Code klar erkennbaren Hauptfunktionen des Bots und ist bewusst praxisnah gehalten, damit das Paket nach dem Einsetzen echter IDs schneller wieder einsatzfähig ist.
