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
  `CFG.bezug`, der gemeinsame Rahmen an `CFG.zuschnitt`. Jeder Feuerlöscher
  hat seine eigene Marke (seit v1.116, vorher gemeinsam je Standort).
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
- **Blattvorschau zoomt selbst (v1.138):** Safaris Kneifen ist auch dort aus
  (es vergrößerte seit v1.94 Kopf- und Fußleiste mit). `blattEinpassen` passt
  den Bogen beim Öffnen in die Breite, zwei Finger ändern `--blattZoom`
  (CSS `zoom` auf den Kindern von `#blattWrap`, bis `BLATT_MAX`); der Anker
  wird beim Aufsetzen am Bogen selbst gemerkt. Kein Doppeltipp (vom Nutzer so
  entschieden). Das geteilte Blatt bleibt unberührt.
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

## Termine (v1.103)

- `TERM` (`nwTermine`, in der Sicherung als `termine`): Verabredungen mit
  Uhrzeit (`zeitVon/zeitBis`) oder `ganz`, `von`–`bis` auch mehrtägig,
  `wdh {n, einheit}`. Kein „erledigt“ – vorbei, wenn die Zeit um ist.
- Vorkommen über `terminVorkommen`; Liste zeigt je Termin das nächste
  (`termineFuerListe`, obenan in der Tagesgruppe über `listenGruppe`).
- `tagKap(t)` = Arbeitszeit minus `terminBelegt`; gilt für Wochenplan,
  Auslastung und „Vorziehen“.
- iPhone-Kalender über eine .ics-Datei (`terminIcs`), wahlweise mit
  Erinnerung
  – bestätigt: nur das Teilen-Menü (`terminTeilen`) funktioniert, direktes Öffnen in Safari hing (v1.104).
- Datumsfelder auf Seiten mit Kopfkarte: übernehmen bei `input`, neu aufbauen
  gar nicht (`datumBinden`, auch nicht bei `blur` – iOS meldet es schon beim Öffnen der Wahl); sonst verrutscht das Feld unter der
  offenen iOS-Datumswahl.

- Wortwahl (v1.106): „Termin“ heißt nur die Verabredung mit Uhrzeit. Bei
  Aufgaben, Mängeln und Wetterregeln heißt das Datum „Fällig am“ bzw.
  „fällig …“ (intern weiter `frist`, `data-tart="termin"`).

- Feldhöhe (v1.107): alle einzeiligen Felder und Auswahllisten haben
  `--feld-h` (40 px seit v1.120, vorher 48), Datum/Uhrzeit ohne iOS-Eigenform, Wert mittig über `line-height` (v1.108; ohne sie saß er unter iOS links oben). Neue Felder nicht mit
  eigener Höhe versehen. Von/bis nebeneinander über `.vonBis`.

- Rahmen (v1.109): Zusammengehöriges mit mehr als einer Zeile steht in
  `.gruppe` (Helfer `gerahmt(an, html)`), nur wenn es gerade mehrzeilig ist.
  Aufgabe, Termin, eigene Wetterregel, Objekt/Prüfer, Feuerlöscher,
  Zählertausch (v1.110). Was schon in einer eigenen Karte steht, bleibt ohne.

- Dunkel (v1.112): Klasse `dunkel` an <html> (dunkelFolgen), nicht per
  Media-Abfrage – sonst würden ausgegebene Blätter dunkel. Neue Farben als
  Variable anlegen und unter `html.dunkel` mit dunklem Wert versehen.

- Ringe der Aufgabenliste (v1.114, vom Nutzer so entschieden): rot =
  überfällig oder heute fällig, gelb = morgen fällig, angefangene Runde oder
  laufende Vorbereitung, sonst keiner. Grün nur in „Alle Nachweise“. Die
  Zeile „Heute auf einen Blick“ (`heuteZeile`) steht unter der Überschrift
  „Heute“, die Gruppe steht immer da.

- Feuerlöscher (v1.116, vom Nutzer so entschieden): jeder Löscher ist ein
  Objekt wie alle anderen – eigene Marke, eigene Zeile, Reihenfolge der Liste.
  Kein gemeinsamer Standort, keine Anzahl, kein „Weiterer Löscher“, kein
  Aufteilen. `info.standort` ist nur Beschreibung; `info.anzahl` aus alten
  Daten wird ignoriert.

- Standort der Feuerlöscher (v1.117): Kacheln unter dem Feld – erst die
  Kurznamen der Grundrisse (`planKurzNamen`), dann die Bauteile
  (`CFG.bauteile`, ab Werk BT I, BT II, einstellbar unter Objekt und
  Prüfer). Der gewählte Plan steht in `info.plan`; `verorten` öffnet ihn,
  solange es keine Marke gibt. Grundriss-Symbol: `ORT_SVG` (Plan mit
  Stecknadel) statt des Zeichens ⌖.

- Knöpfe (v1.118): alle Handlungsknöpfe (`.btn` ohne `.eintrag`, Pfeile,
  Verlauf, Stufen) sind `--feld-h` hoch, Rand `--field-border`, 16 px,
  einzeilig, weiß (nur `.primary` schwarz). Symbolknöpfe: `.quadrat`.
  Listeneinträge (`.eintrag`) wachsen mit dem Inhalt. Neue Knöpfe ohne
  eigene Höhe oder Schriftgröße anlegen; auf 320 px prüfen.

## Testen

Chromium liegt unter `/opt/pw-browsers/chromium-1194/chrome-linux/chrome`,
Playwright ist global installiert (`npm root -g`). Bewährt hat sich:
`index.html` über einen kleinen lokalen HTTP-Server ausliefern, im
Init-Skript `localStorage.nwFrei` auf den Prüfwert aus `const ZUGANG` setzen
(entsperrt die App), Testdaten per `page.evaluate` anlegen und in iPhone-Größe
(390 × 844, `screen` gleich gesetzt) sowie iPad-Größe prüfen. Seitenfehler
über `page.on('pageerror')` sammeln. Vorher die Syntax prüfen:
`new Function(scriptInhalt)` für jeden `<script>`-Block.

- Rückkehr aus dem Grundriss in die Objektliste der Einstellungen (v1.127):
  seit dem Öffnen der Seite neu angelegtes Objekt (`objFrisch`) mit erster
  Marke → Karte zu, Liste oben; sonst Karte offen, gleiche Stelle, gleiche
  Reihenfolge (`objPlanWeg.basis`). Standort-Kacheln rollen nur bei Eingabe
  oder Tipp ins Bild, nie beim Aufbau.

- Einrasten (v1.129): `markeFangen` beim Ziehen einer Marke – gleiche Höhe/
  Senkrechte wie eine Nachbarmarke (`.marke.andere`, also dieselbe Art) und
  gleicher Abstand in einer Reihe, Fang `FANG` px auf dem Schirm, Linien
  `.fangLinie` in Plan und Lupe. Schalter `nwEinrasten` (nur Gerät) unter
  Einstellungen › Grundrisse.
- Wand (v1.131, umgebaut v1.133): `wandFangen` tastet das Planbild
  (`planVorrat`) rund um die Marke in Schirmpixeln ab (`WAND_DUNKEL`). Nur
  Linien, die der Ring berührt (`MARKE_R` = 13 px + `WAND_BERUEHRT`), zählen –
  der Nutzer drückt die Marke an die gemeinte (vom Nutzer so gewünscht).
  Mehrere: tiefer berührte, bei Gleichstand Schieberichtung (`zug.schub`).
  Gerade: Strahlenfächer bis zur ersten dunklen Stelle, Ausgleichsgerade ohne
  Ausreißer (klappt bei Schraffur und Schräge), Länge ≥ 1,5 R; keine
  Dickeprüfung. Bis 5° wird waagrecht/senkrecht genau. Steht die Marke schon
  auf der Höhe ihrer Reihe (`fang.hatX/hatY`), geht die Reihe vor.
  Linie `.fangLinie.wand`. v1.134: Schraffur wird geschlossen (zweimal
  ausweiten/zurücknehmen), Schieberichtung (bis 70°) vor Tiefe; in der Reihe
  rastet sie auch Ring an Ring ein. v1.136 (vom Nutzer so gewünscht): alle
  Marken 20 px, auch die angewählte (hebt sich nur durch Kern, Ring, Puls
  ab); `MARKE_R` = `ANLAGE_R` = 10. In der Lupe ist die gezogene unangewählt.

- Umschalten (v1.130): langer Druck (`UMSCHALT_MS`) auf eine Nachbarmarke
  beim Verorten → `markeUmschalten` setzt `planZiel` auf sie, danach im
  selben Zug ziehen. Bis dahin (`zug.fremd`) kein preventDefault, damit
  Wischen den Plan schiebt; kurzer Tipp zeigt weiter den Namen. Marken tragen
  `data-key`. Nicht in der Übersicht (vom Nutzer so entschieden).

- Sammelmarken (v1.132): `sammelnAuffrischen` fasst in der Übersicht
  Nummernmarken zusammen, sobald eine die Ziffern der anderen verdecken würde
  (`sammelNoetig`, Schirmpixel, seit v1.134; überlappende Kreise allein
  reichen nicht; seit v1.137 zählt nur der farbige Kreis, Halbmesser 11,
  und die Nummernbreite wird per `measureText` gemessen) – auch verschiedene
  Arten. Linie → `.kapsel`
  gedreht (Ziffern über `--gegen` aufrecht), sonst `.kaestchen` mit Zeilen.
  Neu bei Zoom (`planZoomAnwenden`), Überblendung (`markenGewichte`) und
  Aufbau. Tipp nennt alle. Nicht im Blatt (vom Nutzer so entschieden).
