# Adversarial Review — Claude Code / Fable

Modell: `claude-fable-5-1`. Review vom 23. Oktober 2026 auf Grundlage der eingefrorenen Planfassung. Nachfolgend die unveränderte Review-Ausgabe.

# Gesamturteil

Das Produktkonzept ist tragfähig. Die Trennung von Kanzlei, Mandant, Mandat, Projekt, Fenster und Dokument ist richtig, und die Linie „lokal arbeitsfähig, zentral berechtigt" passt zum vorhandenen Code. Ein implementierbarer Plan ist es noch nicht. Zwei Entscheidungen sind offen, ohne die Schritt 3 nicht entworfen werden kann: die Bedeutung von „Speichern" in nativen Sync-Projekten und der Ort der Mandatsidentität. Der Bericht stuft beide als später klärbar ein. Tatsächlich bestimmen sie bereits das Datenmodell von Schritt 1 und den Umfang des Sync-Umbaus. Schritt 1 ist nach einer kleinen Umstellung sofort baubar.

Grundlage: Der Snapshot wurde vollständig gelesen. Zitierte Codezeilen stammen aus dem lokalen Checkout vom 23. Oktober 2026. Dieser kann vom Berichtsstand `bb77ce7` abweichen; ich habe die Gleichheit nicht per Git geprüft. Das Backend `model-api` war nicht einsehbar, serverseitige Aussagen sind Clientvertrag oder markierte Annahme.

# Findings

**1. P1. Der direkte Mandant-Link am Projekt schafft eine zweite Wahrheit neben dem späteren Mandat.** Art: echter Widerspruch zwischen Zeile 195 und Zeile 197.

- Stelle: Abschnitt 5.1, Zeilen 179 und 195 bis 197; Schritt 1, Zeile 323; offene Frage 3, Zeile 361.
- Fehlablauf: Ein Projekt erhält in Schritt 1 genau einen Mandanten. Später kommt das Mandat hinzu, und dieses hat laut Zeile 195 zwei Mandanten, etwa zwei Mitgesellschafter. Unter welchem Mandanten steht das Projekt in der Sidebar, und welche Seite führt bei Widerspruch zwischen Projektfeld und Mandatsbeteiligten? Der Bericht verlangt, Widersprüche zu verhindern, legt aber keine führende Seite fest.
- Zweiter Fehlablauf: Zeiterfassung soll laut Zeile 307 am Mandat hängen. Kommt Schritt 4 vor der sichtbaren Mandatsebene, hängen Leistungen am Projekt. Das Umhängen eines Projekts auf ein anderes Mandat erzwingt dann eine Umbuchung historischer Leistungen.
- Auswirkung: Die versprochene Migrationsarmut gilt nur für den Einmandantenfall. Mehrmandantenmandate, das Zusammenführen zweier Projekte unter einem Mandat und die Finanzsicht brauchen später doch eine Datenmigration.
- Minimale Korrektur: Das Mandat ab Schritt 1 als Datensatz führen, aber unsichtbar. Jede Mandantenzuordnung erzeugt implizit ein Mandat mit einem Beteiligten in der Rolle Mandant. Das Projekt zeigt auf das Mandat, die Sidebar gruppiert nach Beteiligten mit Rolle Mandant. Sichtbar wird das Mandat erst, wenn ein zweites Projekt oder ein DMS-Bezug hinzukommt.
- Beleg: Mandant und Aktenzeichen sind heute Freitextfelder in `packages/types/src/project-templates.ts:87`. Ein Projekt gehört auf der Plattform einer Person laut `apps/server/src/eigenwelt-projects.ts:50`. Annahme: dass die Plattform ein Mandatsobjekt aufnehmen kann, war nicht prüfbar.

**2. P1. Der Zustand „lokal gesichert, noch nicht geteilt" hat in nativen Sync-Projekten keinen Ort.** Art: offene Entscheidung, die der Bericht als Oberflächenfrage behandelt, obwohl sie die Architektur bestimmt.

- Stelle: Abschnitt 6.2, Tabelle Zeile 259 und Absatz Zeile 271; offene Frage 1, Zeile 359.
- Fehlablauf: Der Projektordner ist die Quelle des Syncs. Der DOCX-Editor schreibt per Autosave nach kurzer Verzögerung in genau diese Datei, und eine Sync-Runde startet wenige Sekunden nach jeder Ordneränderung. Autosave ist damit heute schon Veröffentlichung. Dasselbe gilt für Word, Finder und den Agenten, die direkt in den Ordner schreiben. Der einzige nicht geteilte Entwurf ist die Absturzsicherung in IndexedDB des Browserprofils.
- Auswirkung: Der vierte Zustand der Tabelle ist mit der heutigen Architektur nicht herstellbar. Entweder wird Autosave vom Ordner getrennt, oder der Sync hält Dateien in Bearbeitung zurück. Die zweite Variante scheitert an allen Schreibern außerhalb des Legal-Work-Editors. Offene Frage 1 entscheidet deshalb, ob der bestehende Dateisync Quelle bleibt. Ohne diese Entscheidung lässt sich Schritt 3 nicht entwerfen.
- Minimale Korrektur: Für native Projekte festlegen, dass der Ordner Quelle bleibt und Autosave veröffentlicht. „Entwurf" ist dort nur der Editor-Schatten im Konfliktfall. Für DMS-Dokumente ist die vorhandene Arbeitskopie der Entwurf, und Zurückspeichern bleibt ein eigener Schritt. Beide Welten im Bericht ausdrücklich trennen, statt eine gemeinsame Zustandstabelle zu versprechen.
- Beleg: Rundenstart wenige Sekunden nach Änderungen in `docs/project-sync.md:31`; Autosave-Verzögerung in `apps/app/src/react-app/domains/session/artifacts/document-autosave.ts:3`. Absturzsicherung nur in IndexedDB in `docx-recovery.ts:11`; Arbeitskopie und explizites Zurückspeichern in `docs/file-storage.md:47`.

**3. P1. Dokument-IDs und unveränderliche Versionen sind kein „Ergänzen", sondern ein Umbau von Sync, Fensterkoordination und Konfliktlogik.** Art: fehlende Spezifikation und unterbewerteter Aufwand.

- Stelle: Abschnitt 6.4 Zeile 293, Schritt 3 Zeile 333, Messkriterium 7 Zeile 353.
- Fehlablauf eins: Anna benennt im Finder „Vertrag.docx" um, während Ben die Datei bearbeitet. Der Sync kennt nur Pfade und sieht Löschung plus Neuanlage. Bei „gelöscht hier, geändert dort" gewinnt die Änderung. Ergebnis sind zwei Dateien ohne Konfliktmeldung.
- Fehlablauf zwei: Jemand benennt in einem geteilten Projekt einen Ordner mit vielen Dateien um. Der Sync sieht viele Löschungen und hält wegen der Löschschwelle an, bis jemand „Für alle löschen" wählt.
- Fehlablauf drei: Die Version einer Datei ist ihr Inhalts-Hash. Eine Rücknahme auf einen früheren Inhalt ist von „unverändert" nicht unterscheidbar. Clientseitig gibt es weder Versionsnummer noch Autor noch Historie. „Keep mine" ersetzt die Kanzleifassung für alle; ob die Plattform die vorige Fassung behält, ist unbekannt.
- Auswirkung: Die atomare Basisversionsprüfung schützt Hochladen gegen gleichzeitiges Hochladen. Sie liefert keine Versionsreferenz, keine Herkunft und keine Wiederherstellung. Schritt 3 braucht ein ID-basiertes Plattformlisting, einen Pfad-ID-Abgleich je Gerät, neue Schlüssel für Fensterkoordination und Entwurfssicherung sowie Konfliktregeln für Umbenennung. Das ist der teuerste Teil des Plans und steht ohne Aufwandsangabe neben vier weiteren Punkten.
- Minimale Korrektur: Für native Projekte zunächst eine serverseitige, anfügende Versionshistorie je Pfad mit Autor und Zeit sowie Umbenennungserkennung per Hash. Echte Dokument-IDs nur im DMS-Pfad, wo das Quellsystem sie liefert. Messkriterium 7 für native Projekte ehrlich einschränken.
- Beleg: Pfadbasierte Entscheidung in `apps/server/src/project-file-sync.ts:178`, Löschschwelle in Zeile 338 derselben Datei; Version gleich Hash in `apps/server/src/eigenwelt-project-storage.ts:52`. Fensterschlüssel ist der Hash des Pfads in `apps/server/src/routes/files.ts:1180`; Regel „Änderung schlägt Löschung" in `docs/project-sync.md:41`.

**4. P2. Der Plan sagt nicht, wo der Mandantenstamm liegt, und die naheliegende Ablage wäre falsch.** Art: fehlende Spezifikation.

- Stelle: Abschnitt 5.1 Zeile 197, Schritt 1 Zeile 323.
- Fehlablauf: Wird die Zuordnung als Projektdetail gespeichert, läuft sie über den Umfangsschalter „Projektdetails". Hat der Eigentümer diesen Schalter aus, sehen Empfänger das Projekt unter „Ohne Mandant". Details folgen zudem der Regel „neueste Änderung pro Feld gewinnt". Liegt der Stamm nur auf der Plattform, kann im Flugzeug kein Projekt für einen neuen Mandanten angelegt werden, obwohl Abschnitt 6.1 Offline-Arbeit ohne Vorbereitung verspricht.
- Auswirkung: Entweder bricht Messkriterium 1 für Empfänger oder Messkriterium 4 für lokale Arbeit. Lokal erzeugte Mandanten brauchen einen Abgleich beim Hochsync, sonst entstehen Dubletten, die der Bericht erst beim Migrationsassistenten bedenkt.
- Minimale Korrektur: Mandant und implizites Mandat sind Organisationsentitäten mit lokal erzeugten UUIDs und einer Ausgangswarteschlange nach dem Muster der Projekt-Outbox. Die Projektzuordnung ist ein Attribut des Plattform-Projektsatzes wie Name und Zugriff, unabhängig von Umfangsschaltern, mit Kopie in der lokalen Projektdatei. Namensgleiche Mandanten werden zur Zusammenführung vorgeschlagen, nie automatisch verschmolzen.
- Beleg: Umfangsschalter in `packages/types/src/workspace.ts:91`, Sync nur bei Anmeldung in Zeile 143 derselben Datei. Regel „neueste Änderung pro Detail" in `docs/project-sync.md:25`.

**5. P2. Die Aktivitätsanzeige hat weder Transportkanal noch das Wissen, das die erwogene Warnung braucht.** Art: offene Entscheidung plus fehlende Spezifikation.

- Stelle: Abschnitt 6.2 Zeilen 257, 263 und 267; Abschnitt 6.3 Zeilen 279 und 282 bis 283; offene Frage 2.
- Fehlablauf: Der Ereignisstrom der Plattform meldet nur „Projekte oder Aufgaben haben sich geändert". Word-Sperrdateien sind vom Sync ausgeschlossen. Die einzige Anwesenheitsanzeige existiert für laufende Tabular Reviews, als Markierung in der hochgeladenen Datei mit zehn Minuten Lebenszeit. „Matteo bearbeitet" für Dokumente setzt also eine neue Plattformfunktion voraus. „Matteo hat ungespeicherte Änderungen" verlangt zusätzlich, dass der Dirty-Zustand das Gerät verlässt, als Herzschlag oder als hochgeladener Entwurf.
- Zweiter Fehlablauf: Der wichtigste Fall für die Warnung ist dieselbe Person auf zwei Geräten. Der Bericht behandelt nur eigene Fenster und verschiedene Teammitglieder. Zwei eigene Geräte fallen heute unter die Teamregel, und die Konfliktkopie heißt dann „Vertrag (Matteo, Datum)".
- Auswirkung: Ohne Festlegung, was die Anzeige wissen darf und wie lange ein Lebenszeichen gilt, lässt sich weder Warnung noch Sperre entwerfen. Eine harte Reservierung braucht eine Befugnis zum Aufbrechen, also eine Rollenfrage aus Schritt 2, die dort fehlt.
- Minimale Korrektur: Als Standard einen Herzschlag je Dokument mit Dirty-Flag und kurzer Lebenszeit, als Hinweis ohne Garantie. Damit ist die erwogene Warnung ohne Entwurfsupload erreichbar, und die Sperrfrage bleibt ausdrücklich offen. Harte Reservierungen nur als Opt-in je Dokument, nur online erwerbbar, mit protokolliertem Aufbrechrecht für Mandatsverantwortliche.
- Beleg: Ereignisinhalt in `docs/project-sync.md:31`, Review-Lebenszeichen in Zeile 56 und Ausschluss der Sperrdateien in Zeile 46 derselben Datei. Fensterkoordination nur innerhalb eines Browserprofils laut `apps/app/src/react-app/domains/session/artifacts/document-ownership.ts:27`.

**6. P2. Der Agent fehlt als Schreiber im Zustandsmodell.** Art: fehlende Spezifikation.

- Stelle: Abschnitte 6.2 und 6.3 nennen nur Personen; Zeile 265.
- Fehlablauf: Ein Agent überarbeitet eine DOCX, während dieselbe Person sie im Editor offen hat und tippt. Der nächste Autosave scheitert an der Zeitstempelprüfung. Der Editor zeigt ein Banner mit der Aufforderung, eine Kopie herunterzuladen, bevor die neueste Fassung geöffnet wird. Es gibt keinen Vergleich und keine Übernahme, und der Autosave bleibt bis zu einer ausdrücklichen Aktion pausiert.
- Auswirkung: Zeile 269 verspricht „vergleichen / übernehmen" für Änderungen anderer. Für Agentenänderungen gilt heute eine Sackgasse. Der Herzschlag aus Finding 5 wüsste nichts von Agentenarbeit, und Agentenschreibvorgänge umgehen jede spätere Entwurfsschicht.
- Minimale Korrektur: Agentenschreibvorgänge unterliegen derselben Basisversionsregel und erscheinen als Aktivität „Agent für Matteo". Bei aktivem menschlichem Schreiber legt der Agent eine neue Datei oder Version an, statt in die geöffnete Datei zu schreiben. Die bestehende Vergleichskopie ist dafür ein Ansatzpunkt.
- Beleg: Zeitstempelprüfung mit Status 409 in `apps/server/src/routes/files.ts:1319`; Bannerbedingung in `apps/app/src/react-app/domains/session/artifacts/artifact-panel.tsx:714`. Bannertext in `apps/app/src/i18n/locales/en.ts:5718`; Pausieren nach Fehler in `document-autosave.ts:31`.

**7. P2. „Quarantäne und Übergabe" bei Rechteentzug braucht einen Schreibpfad für Entzogene und scheitert an eigenen Ordnern.** Art: fehlende Spezifikation mit Sicherheitsbezug.

- Stelle: Abschnitt 6.3 Zeile 285, Messkriterium 6 Zeile 352, Abschnitt 2.6 Zeile 100.
- Fehlablauf: Heute hält der Client eine Kopie mit ungesendeten Änderungen zurück und fragt „als eigenes Projekt behalten oder entfernen". Ein vom Mitglied selbst eingebrachter Ordner wird nie gelöscht, auch nicht bei Entzug. Die vorgeschlagene Übergabe muss Daten einer Person annehmen, die kein Projektrecht mehr hat, ohne ihr Lese- oder Listenrechte zurückzugeben.
- Auswirkung: Ohne einen nur schreibbaren Ablageort auf Organisationsebene wird die Übergabe zur Rechteausweitung oder bleibt beim heutigen Privatisieren. Für eigene Ordner ist ein Rückruf gar nicht vorgesehen, was Messkriterium 6 für diese Projekte unerreichbar macht.
- Minimale Korrektur: Quarantäne als organisationseigener Ablageort mit Nur-Schreiben für die entzogene Person und Freigabe durch Mandatsverantwortliche. Beim Teilen aus einem eigenen Ordner warnen oder es für kanzleiverwaltete Mandate verbieten. Organisationsbesitz aus Schritt 2 ist Voraussetzung, deshalb gehört dieses Verhalten nicht in Schritt 1.
- Beleg: Zurückhalten bei ungesendeten Änderungen in `apps/server/src/project-sync.ts:599`, Schutz eigener Ordner in Zeilen 559 und 583 derselben Datei. Dokumentiert in `docs/project-sync.md:75`.

**8. P2. Der verwaltete verschlüsselte Cache ist ein zweites Dateimodell und kollidiert mit der Produktbasis.** Art: Widerspruch zwischen Abschnitt 6.5 und Abschnitt 6.1 sowie dem Nutzerwunsch.

- Stelle: Abschnitt 6.5 Zeilen 301 bis 303; Schritt 3 Zeile 333 nennt ihn nicht, Messkriterium 6 setzt ihn voraus.
- Fehlablauf: Projektdokumente sind gewöhnliche Dateien, die Word, Finder, Cloud-Ordner-Erkennung und Agent direkt nutzen. Ein verschlüsselter Cache mit Geräteschlüsseln bedeutet ein virtuelles Dateisystem oder den Verzicht auf externe Bearbeitung. Beides widerspricht der Offline-Arbeit ohne Vorbereitung und der freien lokalen Arbeitsweise, die der Bericht behalten will.
- Auswirkung: Der Aufwand ist nicht bewertet. Die Trennung „persönlich frei, kanzleiverwaltet gesichert" verdoppelt Sync, Editorpfade und Agentenzugriff.
- Minimale Korrektur: Für den ersten Ausbau streichen. Gerätevollverschlüsselung und Rückrufsemantik beim Wiederverbinden dokumentieren und als Grenze benennen. Einen Cache nur für DMS-Arbeitskopien erwägen, wo heute schon eine getrennte Arbeitskopie existiert.
- Beleg: „Dokumente bleiben gewöhnliche Dateien" in `packages/types/src/workspace.ts:21`; Arbeitskopien in einem Projektunterordner laut `docs/file-storage.md:41`.

**9. P3. Der Bericht beschreibt Fensterverhalten, das der Code nicht hat.** Art: Abweichung zwischen Bericht und Implementierung.

- Stelle: Abschnitt 6.2 Zeile 265, Messkriterium 5 Zeile 351.
- Fehlablauf: Laut Bericht beansprucht ein wiederhergestellter Tab keine Schreibzuständigkeit. Tatsächlich fordert jede Ansicht beim Einhängen die Sperre an, wenn sie frei ist. Nach einem Start mit vielen wiederhergestellten Arbeitsbereichen hält der Hintergrund-Tab die Datei, und ein neues Fenster ist Leser und braucht eine Übergabe mit Dialog. Die Alternative „mehrere Fenster teilen eine Editor-Sitzung" existiert nicht.
- Auswirkung: Kein Datenverlust, aber genau die Reibung bei vielen Tabs, die vermieden werden soll.
- Minimale Korrektur: Sperre erst beim ersten Tastendruck erwerben, nicht beim Einhängen, und den Bericht anpassen.
- Beleg: Anforderung beim Start in `apps/app/src/react-app/domains/session/artifacts/document-ownership.ts:150` mit Option „falls frei" in Zeile 58.

# Empfehlungen und Grenzen

**Empfohlene Entscheidungen**

1. Bedeutung von Speichern: In nativen Projekten bleibt der Ordner Quelle, Autosave veröffentlicht, kein zusätzlicher Klick. Entwürfe existieren dort nur als Editor-Schatten und Konfliktkopie. Bei DMS-Dokumenten ist die Arbeitskopie der Entwurf, Zurückspeichern ist bewusst. Begründung: Jede andere Variante bricht Word, Finder und Agent oder verlangt Entwurfsupload.
2. Parallele Bearbeitung: Herzschlag mit Dirty-Flag als Hinweis, keine Sperre beim ersten Ändern. Reservierung als Opt-in je Dokument, nur online, mit Aufbrechrecht. Die vom Nutzer erwogene Warnung ist damit umsetzbar; ob eine Sperre dazukommt, bleibt seine Entscheidung.
3. Mandatsebene: Implizites Mandat ab Tag 1 im Modell, in der Oberfläche unsichtbar, bis ein zweites Projekt, ein Aktenzeichen oder ein DMS-Workspace es braucht. Die verbleibende fachliche Entscheidung lautet: Ein Projekt gehört zu höchstens einem Mandat, ein Mandat hat einen oder mehrere Beteiligte in der Rolle Mandant, interne Projekte haben keines von beiden. „Nach Mandanten" gruppiert nach Beteiligten, „nach Mandaten" nach Akten.
4. Mandantenstamm: Organisationsentität, lokal repliziert mit Outbox und lokal erzeugten UUIDs. Die Projektzuordnung ist ein Plattformattribut des Projekts, nicht ein Detailfeld.
5. Dokumentidentität: Native Projekte bleiben pfadbasiert und erhalten eine serverseitige Historie. Echte IDs nur im DMS-Pfad.

**Erster Umsetzungsschritt, neu geordnet**

- Umfang: Mandantenstamm, implizites Mandat, Projektzuordnung, beide Sidebar-Ansichten, ein Auswahlfeld bei Anlage und Bearbeitung mit den Optionen „vorhandener Mandant", „neuer Mandant", „kein Mandant". Kein Migrationsassistent, kein Beteiligtenmodell über die Rolle Mandant hinaus.
- Ausdrücklich nicht enthalten: Rechteänderungen, Dokumentidentität, Aktivitätsanzeige. Die Oberfläche verspricht keine neue Sicherheitsgrenze.
- Serverseitig in model-api: zwei Tabellen und ein Projektattribut. Lokal: Tabellen plus Outbox. Keine Änderung am Dateisync.
- Parallel dazu ein kurzer Entwurfsspike für Entscheidung 1 und 2, bevor Schritt 3 geplant wird.

**Akzeptanzszenarien**

1. Zwei Projekte desselben Mandanten mit getrennten Teams: Mitglied B sieht unter „Muster GmbH" nur sein Projekt, keine Zähler oder Namen des anderen.
2. Internes Projekt ohne Mandant, mit der ganzen Kanzlei geteilt: Es steht unter „Ohne Mandant", bleibt geteilt, und beim Teilen entsteht kein Pseudomandant.
3. Offline einen neuen Mandanten anlegen und ein Projekt zuordnen, später online: Der Mandant erscheint einmal auf der Plattform. Ein namensgleicher bestehender Mandant wird zur Zusammenführung vorgeschlagen, nicht automatisch verschmolzen.
4. Empfänger mit ausgeschaltetem Projektdetails-Sync sieht die Mandantenzuordnung trotzdem.
5. DOCX in geteiltem Projekt: A tippt mit Autosave, B ändert dieselbe Datei offline und kommt online. Beide Fassungen sind vorhanden, A sieht einen Hinweis mit Vergleich, A arbeitet ohne „Kopie herunterladen" weiter.
6. Datei umbenennen, während jemand anderes sie bearbeitet: genau eine Datei mit beiden Änderungen oder eine Konfliktentscheidung, kein stilles Duplikat.
7. Ordner in geteiltem Projekt umbenennen: keine Löschsperre und keine Neuübertragung aller Dateien.
8. Zwei eigene Fenster, Fenster 1 wiederhergestellt im Hintergrund: Fenster 2 öffnet dieselbe DOCX und tippt ohne Dialog, Fenster 1 übernimmt nicht still.
9. Agent schreibt eine Datei, die ein Mensch offen hat: Der Mensch sieht „Agent hat geändert, vergleichen / übernehmen", der Entwurf bleibt, der Agent überschreibt nicht.
10. Dieselbe Person auf zwei Geräten: Gerät 2 zeigt „Matteo bearbeitet seit 10:42" innerhalb einer Minute. Nach Zuklappen von Gerät 1 verschwindet der Hinweis nach Ablauf, keine Sperre bleibt zurück.

**Was dürfen Anzeigen nie garantieren**

- „Niemand bearbeitet" ist nur „keine Aktivität bekannt". Abwesenheit eines Lebenszeichens beweist nichts, insbesondere nicht offline.
- „Alle Kopien entfernt" nach Rechteentzug. Eigene Ordner, Exporte und externe Arbeitskopien bleiben.
- „Gespeichert" ohne Zusatz, wo lokal gesichert und veröffentlicht verschiedene Dinge sind.
- „Offline verfügbar" für ein Projekt, solange einzelne Dateien noch nie heruntergeladen wurden.

**Was trotz Prüfung standhält**

- Die vier Bedeutungen von „Cloud" und die drei Rollenfragen sind sauber getrennt und code-nah.
- Die Aussage, dass die Route `/checkout` nur eine Arbeitskopie lädt und der Speichervertrag keine Sperroperationen kennt, stimmt. Der Vertrag in `apps/server/src/file-storage/common.ts:22` enthält weder Lock noch Versionsliste noch Anwesenheit.
- Die Darstellung des Rechteentzugs in Abschnitt 2.6 ist korrekt.
- „Ein offener Tab reserviert nichts", „Offline ist kein Pflichtablauf", „Reservierung nur online erwerbbar" und „DMS bleibt führend" sind richtige und konsistente Leitsätze.
- Die Einordnung der Teamkonten mit Sammelanmeldung deckt sich mit `docs/team-storage.md:54`.
- Die Warnung vor Admin-Durchgriff passt zum heutigen Projektvertrag ohne automatischen Adminzugriff.

**Grenzen dieses Reviews**

- Der lokale Checkout wurde nicht gegen `bb77ce7` abgeglichen; Zeilennummern gelten für den Stand vom 23. Oktober 2026.
- Keine Tests ausgeführt, keine Oberfläche bedient. Das Verhalten eines sauberen DOCX-Editors beim Eintreffen einer neuen Fassung habe ich nicht vollständig nachverfolgt; nach dem Code wird die Ansicht ohne eigenen Entwurf vermutlich neu geladen, das ist eine Annahme.
- Versionshistorie, Rechteprüfung und Ereigniskanal der Plattform waren nicht einsehbar. Alle Aussagen dazu sind Clientvertrag.
- Herstellerangaben zu iManage, ADVOKAT, Clio und Actaport wurden nicht neu geprüft.
- Wiki, Suchindex und Kalender wurden nur stichprobenartig gelesen, kein Sicherheitsaudit.
