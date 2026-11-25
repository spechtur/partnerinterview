# CLAUDE.md – Digitales Partnerinterview-Tool

> Projektgedächtnis für Claude Code
> Letzte Aktualisierung: 2026-11-25 (eigenes Repo, PH-Bold)

---

## Projektüberblick

Generisches Partnerinterview-Tool für PHGR-Weiterbildungen: TN wählen aus einem vorbereiteten
Fragensatz (per QR-Link geladen) die gewünschten Fragen aus, führen zeitgesteuerte Interviews
in wechselnden Rollen durch und geben sich abschliessend eine ressourcenorientierte Rückmeldung.
Der Fragensatz-Inhalt ist austauschbar – das Tool selbst bleibt kontextneutral und
wiederverwendbar für andere Einsatzszenarien.

---

## Tech-Stack

- **Frontend:** Vanilla HTML/CSS/JS – zwei Seiten: `presenter.html` (Vorbereitung) und `join.html` (TN)
- **Backend:** keins – Fragensätze als statische JSON-Dateien in `saetze/`. Die Vorbereitung liest das
  Verzeichnis über die GitHub-Contents-API (CORS offen, 60 Anfragen/Stunde); `saetze/index.json`
  ist nur noch Rückfallebene für lokal/offline
- **Übergabe an die TN:** bevorzugt der kurze Weg `join.html?set=slug` – für den Abgleich zählen nur
  die **Fragen**; abweichende Dauer und abweichender Titel reisen als `&t=` und `&titel=` mit
  (QR-Version ~6 statt ~24, vom Beamer aus mühelos scannbar). Nur bei abweichender Eingabe reist der
  Fragensatz deflate-komprimiert im URL-Fragment mit (`join.html#d=…`)
- **Hosting:** GitHub Pages – eigenes Repo github.com/spechtur/partnerinterview, live unter
  https://spechtur.github.io/partnerinterview/ (`index.html` leitet auf `presenter.html` weiter, Query und Hash bleiben erhalten).
  Die tool-box verlinkt nur hierher; ihre alten Pfade `tool-box/partnerinterview/*.html` sind reine Weiterleitungen.
  Die TN-Seite hat bewusst **keinen** Rücklink – geteilte Links führen nicht in die tool-box.
- **Deployment:** Push to main → automatisch live

---

## Datenmodell (Fragensatz-JSON)

Vorschlag für die Struktur eines Fragensatzes – dient als Ausgangspunkt für die Umsetzung:

```json
{
  "slug": "kultur-der-digitalitaet",
  "titel": "Ressourcen in einer Kultur der Digitalität",
  "timer_sekunden": 300,
  "fragen": [
    "Welches digitale Tool nutzt du bereits sicher im Unterricht?",
    "Welche analoge Ressource hilft dir, mit Unsicherheit umzugehen?"
  ]
}
```

David pflegt neue Fragensätze als eigene JSON-Datei im Repo, referenziert über den Slug
in der URL (z. B. `?set=kultur-der-digitalitaet`).

---

## Design-System: PH-Bold (seit 25.11.2026)

- Ganzflächige Farbe pro Zustand über CSS-Variablen `--bg`/`--fg`, Übergang 0.55 s, `meta theme-color` mitführen
- Vorbereitung: Limette `#c7d200` / `#2b2b2b`, Farbband unten (Türkis, Violett, Gelb, Koralle)
- TN-Seite: Auswahl Türkis `#23b2c6` / `#1c1c1c`, Interview Violett `#6169af` / `#fff`,
  Halbzeit-Meldung Gelb `#fbb900`, Endspurt Koralle `#ed6a69`, Fehler Hellgrau `#c6c6c6` (`faerbe()` in `join.html`)
- Typografie: fette Sans in Versalien (DejaVu Sans/Verdana) für Titel und Etiketten, Serif kursiv für Erklärsätze
- Nummernkästchen `01`, `02` … mit abgerundetem Rahmen; Hauptaktion gefüllt, Nebenaktionen als Umriss
- Kein Glas, keine Schatten, keine Verläufe; eine Datei pro Seite, keine externen Ressourcen (ausser QR-API)
- Testen bei 390×800, 1280×720 und 1880×1000

### Meldungen während der Runde

- Vollflächiges Overlay bei Halbzeit und bei einer Minute Restzeit, per Klick zu schliessen
- Halbzeit nur, wenn sie mindestens bei 2:00 liegt (also ab 4 Minuten Rundendauer – sonst zu nah
  an der Schlussmeldung), Schlussmeldung ab 2 Minuten Rundendauer. Die Vorbereitung zeigt den
  Meldeplan unter dem Dauerfeld an; `meldeplan()` dort und `setzeMeilensteine()` in `join.html`
  müssen übereinstimmen
- Der Countdown rechnet aus einem Zeitstempel, nicht mit einem Zähler, und Meldungen lösen beim
  **Unterschreiten** aus – sonst geht die Uhr nach bzw. fällt eine Meldung aus, wenn der Browser
  das Intervall drosselt (Bildschirmsperre, App-Wechsel)
- Beides wird aus `timer_sekunden` abgeleitet und steht **nicht** im Fragensatz – der QR-Code
  wächst dadurch nicht

### Timer-Balken (Spezifikation)

- 16 px hoher Balken, direkt unter der PHGR-Zeile; beide zusammen `position: sticky` am oberen Rand
- Füllstand nimmt linear über die Zeit ab
- Balken in `var(--fg)` auf der Zustandsfarbe; Halbzeit und Endspurt färben die ganze Fläche
- Akustisches Signal beim Ablauf (kurzer, unaufdringlicher Ton)
- Bedienelemente: Start / Pause / Stopp / Restart – keine Zeiteingabe in der TN-Ansicht
- Dauer kommt aus dem Fragensatz-JSON (`timer_sekunden`)

---

## Konventionen

- Alle Texte auf Deutsch (Du-Form für TN-Ansprache)
- Rollenlogik: Es wählt die Person, die **befragt wird** – sie streicht weg, was sie nicht gestellt
  bekommen möchte, und kann eigene Fragen ergänzen. Texte in `join.html`, `presenter.html` und der
  Karte in der tool-box müssen diese Richtung konsistent benennen
- Keine externen CDN-Abhängigkeiten wenn möglich (offline-tauglich)
- Kein Login, keine Nutzerkonten
- Kein Speichern von TN-Daten (Antworten, Notizen) über die Browser-Session hinaus
- Mobile-first, da TN während des Interviews mit dem Smartphone im Raum unterwegs sind

---

## Bekannte Einschränkungen

- v1 hat keine Bibliotheks-Übersichtsseite – Fragensätze werden nur über direkten Link/QR
  mit bekanntem Slug erreicht, von David vorbereitet
- Nach „Interview fixieren" stellt der Button „Neue Runde" die volle Liste wieder her (Rollentausch)
- Fragensätze im Link ergeben dichte QR-Codes (~110 Module); die Vorbereitung sagt das an und
  empfiehlt Vollbild bzw. den Weg über `saetze/`
- Ein neuer Fragensatz muss nur nach `saetze/` gelegt werden, dann steht er in der Auswahlliste;
  «Datei …» lädt jede JSON direkt von der Festplatte, ohne Ablegen
- Lokal geöffnet (`file://`) zeigt der QR-Code auf die GitHub-Pages-Adresse, weil Telefone
  `file://`-Links nicht öffnen können
- Kein Notizfeld pro Frage in v1 – TN notieren ausserhalb des Tools
- Kein Upload-Interface für neue Fragensätze – David pflegt JSON-Dateien direkt im Repo

---

## Verwandte Tools (tool-box)

- Gruppenlos (Gruppengenerator) – Referenz für Struktur und mobile Bedienung
- Sokrates (Lernbegleiter) – Referenz für die PHGR-CI-Umsetzung
