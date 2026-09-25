# Betriebsnachweise – Hinweise für Claude

Die App „Betriebsnachweise“ steckt vollständig in **einer Datei**: `index.html`
(HTML, CSS und JavaScript, keine Abhängigkeiten, kein Build). Sie läuft auf
iPhone und iPad als App vom Home-Bildschirm. Ausgeliefert wird, was in `main`
steht.

## Arbeitsweise (vom Nutzer so gewünscht)

- **Erst lesen und vorschlagen, dann auf Freigabe warten.** Bei neuen Wünschen
  zuerst den betroffenen Code lesen, einen konkreten Vorschlag machen (was sich
  ändert, wie es aussieht, offene Entscheidungen als kurze nummerierte Fragen
  mit eigener Empfehlung) und erst nach Zustimmung umsetzen. Kleine, eindeutige
  Korrekturen (ein Wert, ein offensichtlicher Fehler) direkt erledigen.
- **Auf Deutsch antworten**, knapp und ohne Fachjargon. Der Nutzer ist kein
  Entwickler.
- **Direkt nach `main` übertragen.** Keine Nebenzweige, keine Pull Requests,
  solange der Nutzer nichts anderes sagt. Vor dem Push `git fetch origin main`
  und prüfen, dass es ein Vorspulen ist.
- **Jede Änderung ist eine neue Version:** `const VERSION = 'v1.xx'` hochzählen
  (nach v1.99 geht es mit v1.100, v1.101 … weiter; `BUILD` trägt das Datum)
  und oben im Kopfkommentar unter `ÄNDERUNGEN` einen Eintrag im Stil der
  bisherigen schreiben (Datum, was und warum, aus Sicht der Bedienung).
  Zurückgenommenes bleibt als eigener Revert-Commit in der Geschichte.
- **Vor jedem Push testen** (siehe unten). Wo es etwas zu sehen gibt, Bilder
  mit `SendUserFile` schicken. Ehrlich sagen, was sich hier nicht prüfen lässt
  (echtes iOS-Verhalten, Töne, Fingergefühl).
- Keine Versionsnummern wie „seit v1.55“ in sichtbare Texte der App schreiben
  (nur in Kommentare und das Änderungsprotokoll).

## Code-Konventionen

- Bezeichner, Kommentare und Texte auf Deutsch, im Stil des bestehenden Codes
  (kurze Hilfsfunktionen, Kommentare erklären das Warum, Versionsvermerk in
  Klammern wie `(v1.83)` bei neuen Stellen).
- Text in HTML immer über `esc()` einsetzen.
- Gespeichert wird in `localStorage` (`Store.get/set`) und IndexedDB (`Bilder`
  für Grundrisse, `Historie`). Neue Daten müssen in die Sicherungsdatei
  (`paketUebernehmen` bzw. die Sicherung) und ältere Sicherungen müssen weiter
  einlesbar bleiben.
- `.gitignore` hält Sicherungen (`*.json`), PDFs und Bilder mit echten Daten
  aus dem öffentlichen Repository heraus. Nie echte Daten committen.

## Wichtige Eigenheiten (hart erarbeitet)

- **App-Gerüst (v1.94):** Die Seite selbst scrollt nicht. Kopf- und Fußleiste
  stehen fest, gescrollt wird `#bild`. Bildlauf immer über `bild()`, `bildY()`,
  `bildNach(y)`, `imBild(el)` – nie `window.scrollTo`/`window.scrollY`.
- **Statusleiste** `black` statt `black-translucent`: sonst legt iOS 27 einen
  weißen Schleier an die Oberkante. Nicht zurückstellen.
- **Speicher unter iOS:** Die App vom Home-Bildschirm hat einen eigenen
  Speicher, getrennt von Safari. Symbol entfernen löscht alle Daten – vorher
  immer über Einstellungen › Sicherung sichern lassen.
- **Passwort:** Im Code steht nur ein Prüfwert (`const ZUGANG`, PBKDF2). Ein
  neues Passwort legt der Nutzer über „Prüfwert berechnen“ auf dem
  Sperrbildschirm fest und schickt nur die ausgegebene Zeile; das Passwort
  selbst nie erfragen.
- **Grundrisse:** Marken hängen an `CFG.orte[k|Name]`, ausgerichtete Pläne an
  `CFG.bezug`, der gemeinsame Rahmen an `CFG.zuschnitt`. Feuerlöscher am
  selben Standort teilen sich eine Marke (`loeMarkeTeilen`).
- **pdf.js** wird mit festen SHA-256-Werten (`PDF_SHA`) geprüft und mit
  `isEvalSupported:false` geöffnet. Bei einem Versionswechsel beide Werte
  mitändern.
- Nebenzweige lassen sich aus der Sitzung heraus nicht löschen (403); das muss
  der Nutzer auf GitHub tun. Die Sitzung gibt oft einen Arbeitszweig
  `claude/…` vor, und eine Prüfung am Ende jeder Antwort verlangt, dass er
  übertragen ist. Dann nach `main` **und** auf diesen Zweig mit demselben
  Stand übertragen und dem Nutzer sagen, dass er ihn löschen kann.
- **Ausgegebene Blätter** (`blattDokument`) übernehmen alle Stilregeln der
  App. Regeln für das App-Gerüst (`html,body{height:100%;overflow:hidden}`)
  schnitten das Blatt deshalb auf einen Bildschirm zu (v1.102). `BLATT_FREI`
  hebt das auf, `dateiTeilen` repariert auch ältere Blätter. Wer neue
  Seiten-Regeln für `html`/`body` einführt, muss sie dort mit aufheben.
- **Ausgabe leert den Nachweis** (`delete DAT[k]` in `ausgeben`). Alles, was
  danach noch gelten soll, braucht einen eigenen Vermerk, z. B.
  `DAT.sprinkler.wocheAus` für den erledigten Wochengang (v1.102).

## Aufgabenliste und Planung (Stand v1.102)

- Gruppen nach Tag (`aufgaben()`, `gruppeVon`): Überfällig, Heute, Morgen,
  Diese Woche, Nächste Woche, dann je Monat. Überfällig nach Wichtigkeit,
  sonst streng nach Tag, dann Wichtigkeit.
- Eingeplant: `a.geplant` (ISO-Tag) an der Aufgabe; ein verstrichener Tag
  fällt still weg (`einplanungAufraeumen`). Feld „Eingeplant für“ auf der
  Aufgabe, der Wochenplan legt Eingeplantes auf seinen Tag.
- „Gute Gelegenheit“ (`gelegenheiten`): Aufgabe mit Wetterprofil, Termin nach
  dieser Woche, guter Halbtag in den nächsten 3 Arbeitstagen und genug freie
  Zeit laut `wochenplanRechnen(liste).freiVor`. „Nicht jetzt“ merkt
  `a.gelegenheitNein`. Nur bei gutem Wetter – so vom Nutzer gewünscht.
- „Zeit übrig? Vorziehen“ (`vorziehenRechnen`): freie Zeit heute =
  `heuteRest()` minus Überfälliges und Heutiges; Vorschläge ab Morgen, nichts
  Ungesehenes außerhalb der Liste (vom Nutzer so entschieden).
- Arbeitsmittel je Objekt: `CFG.mittel[Schlüssel] = {ut, mat}`, Schlüssel wie
  bei Mängeln (`k|Anlage`, `medien|Zählerschlüssel`,
  `sprinkler|woch|Punkt`). Eingetragen im Nachweis an der Karte, bewusst nicht
  in den Einstellungen. Fließt in Listenzeile, Vorbereiten/Material je Runde
  und in Mängel; `mittelUmhaengen` beim Umbenennen. Gelöschte Objekte
  behalten ihre Angaben wie ihre Grundrissmarke.

## Testen

Chromium liegt unter `/opt/pw-browsers/chromium-1194/chrome-linux/chrome`,
Playwright ist global installiert (`npm root -g`). Bewährt hat sich:
`index.html` über einen kleinen lokalen HTTP-Server ausliefern, im
Init-Skript `localStorage.nwFrei` auf den Prüfwert aus `const ZUGANG` setzen
(entsperrt die App), Testdaten per `page.evaluate` anlegen und in iPhone-Größe
(390 × 844, `screen` gleich gesetzt) sowie iPad-Größe prüfen. Seitenfehler
über `page.on('pageerror')` sammeln. Vorher die Syntax prüfen:
`new Function(scriptInhalt)` für jeden `<script>`-Block.
