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
  weißen Schleier an die Oberkante. Nicht zurückstellen. Dazu gehört: kein
  `theme-color`, und `header` behält `position:sticky;top:0`, auch wenn im
  festen Gerüst nichts klebt – Safari färbt den Streifen hinter der Uhrzeit
  nach dem Element, das oben klebt (im E-Assistenten v1.132 gelernt: ohne
  `sticky` kam der Schleier zurück). Neu anlegen des Symbols ist dafür nicht
  nötig.
- **Tastatur (v1.180):** `hoeheBinden` setzt `--app-h` = visualViewport und
  holt das Feld über `feldZeigen` hoch (auch bei `focusin`, also beim
  Feldwechsel mit offener Tastatur; in `.dlgbox` rollt das Fenster selbst).
  Auf Touch-Geräten hängt während der Eingabe `.tastaturPlatz` (halbe
  Bildschirmhöhe) unten an `#bild`, weil iOS ein Feld auf kurzen Seiten sonst
  nicht hochrollen kann (Jean); weg bei `focusout`. `#dlg` ist so hoch wie
  `--app-h`. Im Testbrowser nur nachstellbar, indem `visualViewport.height`
  überschrieben und `resize` ausgelöst wird.
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
- **Zoom im Grundriss (v1.142):** Ablauf wie seit jeher (Anker `mx/my` beim
  Aufsetzen, je Fingermeldung sofort gerechnet – ein Umbau mit
  requestAnimationFrame und relativem Anker, v1.139, nahm das Schieben
  während des Zoomens und wurde zurückgenommen). Gegen das Wackeln: Größe
  ungerundet, `mx` ohne den Mittig-Rand, und der Rollrest, den iOS auf ganze
  Pixel schneidet, als `translate` am `#planBox` (auf Bildpunkte gerastet).
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

- Künftige Sprinklergänge (v1.143, vom Nutzer so entschieden):
  `sprinklerVorschau(bis)` liefert listenartige Einträge (`vorschau:true`)
  für jeden späteren Wochengang (an seinem Freitag, bei Feiertag/Abwesenheit
  `wirksamerTag` davor) und jede spätere Monatskontrolle (letzter Arbeitstag).
  Sie zählen still in den Gruppensummen (die Zeile „dazu Sprinkler-…“
  darunter hat der Nutzer in v1.144 abgelehnt), in
  `planPruefen` und `wochenplanRechnen`, stehen aber nicht als Zeilen in der
  Liste. Gruppen entstehen dafür eigens nur bis nächste Woche/Monatsende.

## Anlagentypen (v1.147, Etappe 1 von 2)

- `VORLAGEN` = alle fertigen Typen; `NACHWEISE` enthält nur die in
  `CFG.typen` (Reihenfolge der Vorlagen), neu gefüllt von `typenAnwenden`.
  `NW(k)` findet jede Vorlage (auch entfernte), `aktiv(k)` sagt, ob sie in
  Gebrauch ist – für Existenzprüfungen (`akkuAn`, `loeSumme`, Kategorien
  über `kategorien()`) immer `aktiv` nehmen.
- `typenPruefen`: fehlt `CFG.typen`, neue Installation → leer, sonst
  `typInGebrauch` (Daten, EXP, BUCH, Marken, Objekte ≠ Beispiel ab Werk).
  Auch in `paketUebernehmen` für ältere Sicherungen.
- `typWahl`/`typHinzufuegen` (nie benutzt → leer starten, `typFrisch` öffnet
  das Feld fürs erste Objekt), `typEntfernen` (Daten bleiben).
- Baukasten (v1.148): eigene Typen in `CFG.eigen` (`eigen:true`, Kennung
  `e…` aus Uhrzeit + Zufall), `setTypBau` (Seite `typbau`), `typAbleiten`
  leitet Überschrift/Dateiname ab (`titelEigen`/`dateiEigen` = eigene
  Eingabe). Arten: '' (eigene Kontrolle), 'fremd', 'bestand'. „Nur Bestand“
  steht in `BESTAND`, nicht in `NACHWEISE` (keine Aufgaben/Nachweise); Liste
  aller aktiven: `TYPEN_AKTIV` (Menü, Suche). Etagen über `n.etagen`.
- Zustände: `n.status` frei, `n.mangel` = Zustände, die als Mangel zählen;
  `istMangel(n, st)` (Vorlagen ohne `n.mangel`: Wort „Mangel/defekt“).
- Abstände (v1.148): `taktVon(n)`, Zeiträume `zrVon/zrAnfang/zeitraumEnde/
  zeitraumText/zeitraumVoll` für `2026-W41`, `-10`, `-Q4`, `-H2`, `-J`;
  `expZr`, `restZr`, `TAKT_ROT`. Sprinkler rechnet weiter für sich.
- Nicht im Baukasten: Rundgang (nur die Vorlagen in `RUNDGANG_TYPEN`).

## Akkutausch (v1.149, allgemein)

- Akkutypen: `CFG.akkuTypen` = [{name, monate}] (gemeinsam; `akkuTypenPruefen`
  übernimmt die alten RWA-Typen in Jahren). `akkuMonate`, `fristM`,
  `monateText`.
- Je Objekt: `AKKU` (Store `nwAkku`, Sicherung `akku`) unter `k|Objekt` mit
  `akkuTyp/akkuAnzahl/akkuDatum/akkuSeit`; `akkuZeile` liest, `akkuZ` legt an.
  `akkuAusDat` holt die alten Felder aus `DAT.rwa.zeilen` (Update und alte
  Sicherungen). Nicht in DAT, weil die Ausgabe eigener Kontrollen DAT leert.
- `akkuAktiv(k)`: eigene Typen `n.akku`, Vorlagen `CFG.teile[k].akku`
  (Schalter auf der Typseite) oder Vorgabe `n.akku` (RWA). Nie Sprinkler/Medien.
- Ansicht `akku` je Typ (`akkuK`, `akkuOeffnen(k)`, Titel `akkuTitel`).
  Tätigkeit/Material-Schlüssel `akkuTaet(k)` und Anlass-IDs `akkuAnlassVor`:
  die RWA behält ihre alten (`akku`, `akku:<Objekt>@`), sonst mit Typ.
- Markenfarbe: `ortStatus` = knappere Lage aus `ortStatus0` und Akkufrist.
- Prüfabstand der Fremdprüfungen seit v1.151 in Monaten: `teilCfg(k).monate`
  (aus altem `jahre` umgerechnet), Fristen über `fristM`, Gruppen über
  `monateTitel`. `frist(datum, jahre)` gibt es nur noch für Altes.

## Belegfotos der Zähler – entfernt (v1.156)

- v1.152 brachte ein Belegfoto je Zähler, v1.156 nahm es wieder heraus (Jean:
  Ablesung per Kamera vorerst ausgesetzt, Aufwand zu hoch). Die IndexedDB
  `nwBilder` bleibt auf Fassung 4 mit dem leeren Bereich `belege` – eine
  niedrigere Fassung ließe sich auf den Geräten nicht mehr öffnen.
  `belegeLeeren` löscht einmal, was noch darin liegt.

## Störungen (v1.153)

- `STOER` (Store `nwStoer`, Sicherung `stoerungen`): {id, k, anlage, datum,
  was, weg ('akku'|'reset'|'teil'|'firma'|'offen'), vermerk, aufg, erledigt,
  loesung, akkuAlt, akkuMonate, akkuGesetzt}. Nicht in DAT (Ausgabe leert es).
- Eingetragen nur im Verlauf (`verlaufStoer`/`stoerVerdrahten`, Karte obenan,
  nicht bei Zählern); erreichbar über Uhrsymbol oder Suche (vom Nutzer so
  entschieden, kein eigener Weg von der Startseite). Nicht auf dem Blatt.
- „Akkutausch“ setzt einmal je Störung das Tauschdatum (`datumAbloesen`) und
  merkt das Alter des alten Akkus. „Noch offen“ → eigene Aufgabe mit
  `a.stoerung`; `stoerAusAufgabe` bei erledigt/wieder/löschen.
- Gehäuft ab `STOER_GEHAEUFT` (2) in 12 Monaten: `stoerHinweis` an den Karten,
  Übersicht › Störungen (`renderStoerungen`). `anlageUmbenennen` zieht mit.
- Dauer (v1.154): `s.dauer` über `dauerWahl` im Feld; jede Störung steht in
  `arbEintraege` (Quelle `stoer`), auch ohne Dauer (Jean). Nicht in die
  Durchschnitte (`TAET`) – geplante Akkuwechsel an Nachbaranlagen gehen je
  Anlage viel schneller als einer bei einer Störung.
- Nur ein Eintrag je Störung (v1.155, Jean): Aufgaben mit `a.stoerung` fehlen
  in `arbEintraege`, ihre Dauer zählt über `stoerDauer` zur Störung; „Ändern“
  setzt die Gesamtdauer an die Störung und nimmt sie von der Aufgabe.

## Abläufe: Pläne aus Stufen (v1.159, vorher v1.157/158)

- Jean: Aufgabe prüfen → Ergebnis wählen → je Ergebnis eine Folge, auch
  zurück („in Ordnung → in 90 Tagen erneut prüfen“, „Handlungsbedarf →
  Firma beauftragen“ …). Darum zentrale Pläne in `CFG.ablaeufe` (reist mit
  der Sicherung): `{id, name, stufen:[{id, titel, wichtig, notiz,
  schritte:[{id, art 'link'|'tel'|'mail'|'haken', text, url|nummer|an/
  betreff/inhalt}], ergebnisse:[{id, text, ziele:[{stufe, tage}]}]}]}`; erste
  Stufe = Anfang, Ergebnis ohne Ziele = Ende. Änderungen gelten ab der
  nächsten Stufe (Jean).
- An der Aufgabe: `a.plan`, `a.stufe`, `a.kette` (id der ersten Aufgabe),
  `a.ablaufStand`, `a.ergebnis`/`a.ergebnisText`, `a.planFolgen` {Ziel-Nr:
  id|'nein'}. Verlauf der Kette = ihre erledigten Aufgaben (`ketteVon`).
- Karte auf der Aufgabe (`ablaufKarteHtml`/`ablaufVerdrahten`): „Ablauf
  starten“ (`ablaufStarten`: Plan wählen oder neuen aus der Aufgabe), Schritte
  als echte `<a href>` (iOS), „Beim Erledigen“, „Bisher“. Platzhalter
  {Datum} {Objekt} {Titel}.
- Erledigen: `ablaufErledigen` – Ergebnis per `wahlFenster` (eines → direkt,
  „Ohne Folge“), je Ziel `folgeStufeFragen` mit Rückfrage und Datum (Jean);
  Objekt und Kette reisen mit. Wiederkehrende behalten Plan und Stufe.
- Seiten `ablaeufe` (Übersicht › Abläufe), `ablaufplan`, `stufe`.
- Verlauf des Objekts: Abschnitt „Abläufe“ (`verlaufAblauf`).
- `ablaufUmwandeln`: alte `a.ablauf` (v1.157/158) → Plan (erste Frage →
  Ergebnisse, „immer“-Folge zu jedem Ergebnis), beim Start und in
  `paketUebernehmen`.
- Bedienung v1.160 (Jean: „unübersichtlich“): Begriffe „Aufgaben“ und
  „Hilfen“ statt Stufen/Schritte. Erledigen = Tipp auf ein Ergebnis
  (`ergebnisseHtml`/`ergebnisseVerdrahten`, ✎ ändert das Datum vorher,
  `planWeiter` legt die Folgen sofort an – keine Rückfrage mehr, Jean),
  „Erledigen ohne Folge“ = `erlSetzen`. Ganzer Ablauf auf einer Seite
  (`renderAblaufPlan`, Sätze „Wenn … → … · in 3 Monaten“), Ergebnis im Fenster
  `ergebnisFenster` mit Kacheln „Dann folgt“ und „Wann“ (`WANN`; Ziel trägt
  `tage` oder `monate`, `zielDatum`/`wannText`). `planStarten` (Objekt,
  Datum), „Laufend“ unter Übersicht › Abläufe, Liste „Dachrinne · 2 von 3“.
- Aufgabe aufs Wesentliche (v1.161, Jean: „was vordergründig geht, geht
  unter“): Kopf nur Termin, Objekt, Ablauf („Dachrinne · Aufgabe 2 von 3“),
  Notiz, kleine Knöpfe Grundriss/Nachweis; darunter Wetter (nur
  wetterabhängig), Vorbereitung (nur wenn offen, `aufgVorbOffen`), Hilfen
  (`ablaufHilfenHtml`), Erledigen (Vermerk/Dauer in `.erlMehr`). Alles
  Übrige unter `#aufgMehr` („Mehr: Leiter · ≈ 30 min …“, `aufgMehrText`):
  `ablaufMehrHtml` (Bisher, bearbeiten, „Ablauf starten“), Werkzeug-Karten,
  `aufgInfoZeile`, Angaben, Löschen. `mehrAuf` hält es offen.
- „Gleich im Anschluss“ (v1.161): `s.sofort` an der Aufgabe im Ablauf
  (Schalter auf der Stufen-Seite, „⚡ sofort möglich“ auf der Ablaufseite).
  `planWeiter.neue` sammelt die eben angelegten Folgen; `sofortFenster`
  zeigt Hilfen und Ergebnisse, erledigt erst mit Tipp aufs Ergebnis (Jean:
  manuell bestätigen), „Als Aufgabe speichern“ als leiser Knopf; Kette geht
  weiter, wenn die nächste Folge wieder sofort möglich ist.
- Ablaufseite zugeklappt (v1.161): Kette aus Kärtchen, `abAuf` merkt die
  offenen; „Bearbeiten“ öffnet die Stufen-Seite, ↑ ↓ daneben ordnen
  (v1.162, statt „Weiter nach vorn“ auf der Stufen-Seite – Jean: deplatziert).
- „Datum“ bei Wann (v1.163, Jean: jedes Mal ein anderes – nicht im Ablauf
  festlegen): Ziel `{stufe, wahl:true}` (altes `{datum}` aus v1.162 zählt
  gleich, `datumWahl`). Seit v1.164 steht das Datumsfeld direkt unter dem
  Ergebnis (`zielDatumFeld` über dem Knopf via `ergebnisBlockHtml`, gelesen von
  `zielFelderLesen`), auf der Aufgabe
  und im `sofortFenster` – das Zwischenfenster mit „Abbrechen“ war
  missverständlich (Jean). Ergebnis ohne Namen = „Erledigt“ (`ergName`).
- Fenster „Weiter“ (v1.165, Jean): unten Abbrechen/Speichern (`#dlgNein`/
  `#dlgJa`), Ergebnis bei mehreren als Kacheln (`data-serg`), Datumsfelder
  wie auf der Aufgabe („Später erledigen“ fiel in v1.166 weg, Jean).
  Abbrechen = `sofortZurueck(rueck)`: Stand vor dem Erledigen
  (`vorher`, ganze Kopie) zurück, unberührte neue Folgen und Wiederholung
  weg, dann Seite der Aufgabe bzw. voriges Fenster (`rueck.dann`).
- Dauer im Ablauf (v1.167, Jean): Tätigkeitsschlüssel `ablauf:Plan:Stufe`
  (geht in `taetSchluessel` vor `wdh`). Erledigen setzt `a.dauerOffen`
  (auch im `sofortFenster`), kein `taetFragen`. `ablaufDauerFragen(a)` läuft
  am Ende von `weiterSofort` (nicht bei Abbrechen): steht in der Kette heute
  noch etwas offen an, wartet es; sonst `dauerVermerken` für Handeingaben,
  Fenster mit einer Zeile je Schritt (− +, „wieder fragen“ je Schritt),
  „Später“ nimmt nur `dauerOffen` weg (dann zählt der Schnitt).
  `stufeDauerHtml` auf der Stufen-Seite.
- Arbeitsnachweis (v1.168, Jean): `arbAblaeufeBuendeln` fasst Schritte
  (`aufg` mit `a.plan`, `taet` mit `ablauf:`-Schlüssel; Dauer-Einträge tragen
  seit v1.167 `aufg`) je Tag, Plan und Objekt zu `quelle:'ablauf'` mit
  `teile` zusammen. Ändern = `arbAblaufAendern` (je Schritt), Löschen legt
  alle Teil-Ids in `TAET.arb.weg`.
- Dauerstufen (v1.169, Jean): `DAUER_STUFEN` beginnt mit 5 und 10 min,
  dann 15-min-Schritte bis 4 h, dann Tage; `alsStufe` rundet unter 12,5 min
  auf 5/10. Die Planung (`auf15`) rechnet weiter in Viertelstunden.
- Testskripte im Notizordner überschreiben `ablaufDauerFragen` mit einer
  leeren Funktion, sonst blockiert das Dauerfenster ältere Abläufe.
- **Seitennamen eindeutig halten:** v1.159 nannte die Ablaufseite `plan` mit
  `#planBox` – das ist die Grundriss-Seite. `go('plan')` zeichnete dann den
  Ablauf in den Grundriss, Kopfzeile und Geschoss-Regler fehlten. Seit
  v1.160 `ablaufplan`/`#ablaufPlanBox`, Klassen `ab…`. Vor neuen Seiten,
  Ids und Klassen nach dem Namen suchen.
- Etappe 2 (vereinbart): Runden der Nachweise, Sprinkler, Akkutausch,
  Wetterregeln; selbsttätige Bedingungen (z. B. Mangel in der Runde).

## Online-Dienste (v1.170)

- Gruppe „Online-Dienste“ in den Einstellungen (Name von Jean): Adresse,
  Wetter, Abfallkalender. `CFG.adresse` {strasse, hnr, plz, ort, zusatz,
  lat, lon} über `setAdresse`; Suche bei Photon (komoot, OSM, kennt
  Hausnummern), sonst Rückfall auf die Ortssuche von Open-Meteo. Felder von
  Hand korrigierbar (Jean), die Lage ändert sich nur über die Suche.
  `adresseSetzen` setzt `CFG.wetter` mit (Wetter-Code liest weiter
  `CFG.wetter`); `setWetter` zeigt nur noch den Standort und „Adresse ändern“.
- Abfallkalender: vorerst nur Information, keine Aufgaben (Jean). `ABFALL`
  (Store `nwAbfall`, Sicherung `abfall`, `abfallPruefen`): {url, quelle
  'datei'|'link', datei, abgerufen, versuch, fehler, termine:[{tag, art}]}.
  Datei öffnen (`abfallDateiLesen`, entfernt den Link) oder Link
  (`abfallAbrufen`, beim Öffnen höchstens täglich; TypeError online =
  'gesperrt', also CORS). `icsLesen`/`icsWiederholen`: RRULE, EXDATE,
  RECURRENCE-ID, UTC → Ortszeit. Seite `abfall` (Übersicht › Abfall).
- Ziel „möglichst viele Kommunen“: Weg ist „Link kopieren“ (Knopf der
  ics-Datei auf der Abfallseite lange drücken – antippen gibt sie unter iOS
  nur an den Kalender). Jeans Test (v1.170): api.abfall.io lässt den Abruf
  aus der App zu. Geführte Auswahl bei abfall.io zurückgestellt (Jean): Schlüssel je
  Entsorger, Auswahl-Schnittstelle (GraphQL, eigener Kopf) prüft vermutlich
  die Herkunft. Ein Vermittlungsserver nur, wenn Datei und Link nicht reichen.
- Fester Zeitraum im Link (v1.171): `abfallUrls` setzt in `timeperiod`/
  `f_zeitraum` das laufende Jahr ein, ab November (`ABFALL_NEUES_JAHR`) dazu
  das nächste (Fehlschlag dort zählt nicht). Gespeichert bleibt der Link wie
  eingefügt.

## Aktionen mit Daten (v1.172)

- „Hilfen“ heißen sichtbar „Aktionen“ (Jean); im Code weiter `schritte`,
  `hilfenKnopf`, `ablaufHilfenHtml`. Neue Arten `wetter`/`abfall`
  (`AKTION_INFO`): keine Knöpfe, sondern `aktionInfoHtml` mit dem Stand von
  jetzt. Wetter trägt `x.bed` (Bedingung wie die Wetterregeln, gerechnet mit
  `bedPruefen`/`hinweisReihen`), `x.grenze` → ✓/✗. Abfall trägt `x.arten`
  (leer = alle). Editor: `wetterAktionFelder`/`abfallAktionFelder`.
- Wann „vor Abholung“: Ziel `{stufe, abfall, vor}` (`vorAbholung`,
  `abholungVor`, in `zielDatum`/`wannText`); erste Abholung der Art, deren
  Tag minus `vor` ≥ Basis; sonst die Basis selbst. Kein Arbeitstag →
  `wirksamerTag` davor, nie vor der Basis (v1.173, Jean).
  Auf der Aufgabe steht hinter dem Vorschau-Datum die Regel (`wannText`,
  v1.176) – Jean hielt das Datum sonst für fest. Wiederholen „alle X
  Wochen ab Abschluss“ baut man mit Folgen zurück auf die erste Aufgabe
  („nicht voll → Füllstand prüfen in 2 Wochen“), keine eigene Funktion (Jean).
  „in X Tagen/Monaten“ → `arbeitstagAb` (nächster Arbeitstag danach, über
  `freiGrund`; v1.177, Jean), „sofort“ bleibt der Tag, „vor Abholung“ geht
  auf den Arbeitstag davor.
- Auslöser (v1.174, Jean): die Wetterregeln heißen sichtbar „Auslöser“,
  Liste `renderWetterregeln` in `#ausloeserBox` auf Übersicht › Abläufe (auf
  der Wetterseite nur ein Verweis). Intern weiter `CFG.wetterEigen`,
  `regel…`, `HINWEISE`. Größe `abfuhr` (quelle 'abfall', `b.abfall`, Fenster
  `b.tage` ab heute) liefert `ereignis` = Abholtag → Schlüssel je Abholung.
  `r.dann` 'aufgabe'|'ablauf' mit `r.plan` (`hinweisAufgabe` legt die erste
  Stufe an), `dannText`. Neue Auslöser `modus:'vorschlag'`. Ohne Wetter
  prüfen nur die eigenen (`mitWetter`). `hinweisFrist` 'vorher' →
  `wirksamerTag`. Karte auf der Startseite: „Vorschläge“.
- Achtung: `eigeneRegeln()` gibt eine Abbildung zurück – nie darauf `push`en
  (bis v1.173 gingen so neue eigene Regeln verloren), sondern
  `CFG.wetterEigen`.
- Ergebnis mit Bedingung (v1.175, Baustein 3): `e.bed` (alle müssen
  zutreffen, Bausteine wie bei Auslösern inkl. Abfuhr), `e.auto`
  'vorschlag'|'sofort'. `ergebnisTreffer(a)` nur für fällige Aufgaben
  (Frist ≤ heute), erstes zutreffendes Ergebnis, ohne `a.ergebnisNein`.
  `ergebnisseAnwenden` (sofort, beim Aufbau der Startseite nach den
  Auslösern), `ergebnisVorschlaege` (in der Karte „Vorschläge“, Schlüssel
  `erg:<Aufgabe>:<Ergebnis>`), `ergebnisAusfuehren` (Vermerk „Selbst
  gewählt: …“, `wdhFolgeAnlegen`, `planWeiter`). Editor: Karte „Bedingung“ (v1.178, vorher
  „Selbst wählen“) im `ergebnisFenster` über `bedFelderHtml`/`bedFeldSetzen` (auch die
  Wetter-Aktion nutzt sie).

## Abläufe neu geordnet (v1.179, Jean: „nicht selbsterklärend“)

- Sichtbar „Schritt“ statt „Aufgabe“ im Ablauf (intern weiter `stufen`,
  Seite `stufe`, Titel „Schritt“); „Aufgabe“ = nur, was in der Liste steht.
- `renderAblaufPlan`: Name, „So läuft es ab“ (`ablaufSatz`), „Beginnt“
  (von Hand + `ablaufAusloeser(p)` = eigene Regeln mit `dann:'ablauf'`,
  `#trigNeu` legt einen Auslöser-Entwurf mit diesem Plan an), Schritte immer
  offen (`.abSchritt`, Kopf `data-stbearb`, Ergebniszeilen `.abErg` mit
  `data-st/data-e`), „+ Schritt“, „Starten“, „Ablauf löschen“. Kein `abAuf`,
  keine ↑↓ mehr; der Anfang über „Mit diesem Schritt beginnen“ (`#stufeVorn`).
- `ergebnisFenster` als drei Fragen; im Fenster je Ziel `modus`
  'gleich'|'nach'|'abh'|'datum' (+ `n`/`einh` Tage|Wochen|Monate), gespeichert
  wie bisher {tage|monate|abfall+vor|wahl}. Zugleich-Folgen und Bedingung
  (`#bedAuf`) eingeklappt. `WANN` wird nicht mehr angezeigt.
- `renderStufe`: Titel, „Infos“ (wetter/abfall, `istInfo`, `#infoNeu`),
  „Aktionen“ (link/tel/mail/haken), Notiz, `<details>` „Mehr“
  (Wichtigkeit, Dauer, sofort). Jean lehnte „Zur Hand“ ab: der Name soll zu
  dem passen, was darunter steht. Auf der Aufgabe kommen Infos zuerst.
- „+ Neuer Ablauf“ (`ablaufNeu`): leer oder `ablaufBeispiel()` „Regelmäßig
  prüfen“ (Jean: ein Beispiel genügt).
- Übersicht › Abläufe: Auslöser nur für einzelne Aufgaben (an einen
  vorhandenen Plan gebundene stehen beim Ablauf), Wetter ab Werk in
  `<details>` (`werkAuf`).

## Suche (v1.145)

- `suchObjekte`: Text je Objekt = Name (+ Löscherangaben) + Name der Anlage
  (`n.label`) + Grundrisse seiner Marke (`suchPlanText`: Name und Kurzname aus
  `suchPlaene`, beim Löscher auch `info.plan`). Arbeitsmittel (`o.mittel`) und
  Bemerkung zählen mit und erscheinen in der kleinen Zeile, wenn ein Wort nur
  dort steht. Alle Wörter müssen vorkommen, verteilt auf all das.
- `suchPlaene` kommt verzögert aus `Bilder.alle()`, neu bei jedem Antippen des
  Feldes. Sprinklerpunkte und Checklisten bleiben außen vor (vom Nutzer
  vorerst so entschieden).
- Eigene Treffer (v1.146): „Listen“ (`suchListen`: Werkzeug, Material,
  Akkutypen → `setOeffnen('werkmat'|'akkus')`) und „Grundrisse“
  (`suchGrundrisse` aus `suchPlanListe` → `planZeigen`), unter den Objekten.

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
  Linien, die der Ring berührt (`MARKE_R` + `WAND_BERUEHRT`), zählen –
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

## Offen / bekannt (Stand v1.142)

- iPad mit Tastatur-Trackpad: Kneifen erreicht die Seite nicht (iPadOS
  behandelt es selbst) – weder Grundriss noch Blattvorschau zoomen damit.
- Sammelmarken nur in der App, das Blatt hat seine eigene Beschriftung
  (`markenLegen`); Übernahme ins Blatt wäre ein möglicher nächster Schritt.
- Wanderkennung an echten Plänen nur vom Nutzer geprüft (Schraffur, Schräge,
  parallele Linien klappten nach v1.134); bei neuen Problemen Bildschirmfoto
  erbitten.
