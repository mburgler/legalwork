# Kompakte Zusammenfassung des Plans

**Planstand:** 23. Oktober 2026. **Als Notiz archiviert:** 31. Oktober 2026.

Legal Work soll eine Kanzleistruktur über den bestehenden Projekten erhalten: Mandanten und Mandate, passende Team- und Finanzrechte sowie verlässliche Zusammenarbeit mit lokalen Dateien und DMS. Die Mandantenübersicht kommt zuerst; dafür wird nicht vorab ein komplettes DMS gebaut. Dies ist ein Entwurf für das Zielsystem, keine Beschreibung bereits fertiger Funktionen.

## 1. Mandanten, Mandate und Projekte

Mandant = Person/Unternehmen; Mandat = Auftrag/Fall; Projekt = Arbeitsbereich. Ein Mandant kann mehrere Mandate und Projekte haben, ein Mandat mehrere Projekte und auch mehrere Mandanten. Ein Projekt gehört höchstens einem Mandat. Die Mandatsidentität existiert von Anfang an, wird im einfachen Fall aber automatisch angelegt: keine doppelte Pflichtmaske. Interne Projekte bleiben ohne Mandanten und Mandat möglich – auch als gemeinsam genutzte Kanzleiprojekte.

## 2. Sidebar als persönliche Ansicht

Umschaltbar zwischen Projektliste, Gruppierung nach Mandanten und Gruppierung nach Mandaten. Dieselben Projekte behalten ihre IDs und Speicherorte. Bei mehreren Mandanten können mehrere Verweise auf dasselbe Projekt erscheinen. Zusammenlegen oder Umhängen verändert weder still die Zugriffsrechte noch rückwirkend bereits gebuchte Leistungen.

## 3. Echte, offline nutzbare Stammdaten

Mandanten und Mandate sind keine Freitextfelder und hängen nicht vom optionalen Projektdetails-Sync ab. Neue Datensätze erhalten auch offline eindeutige IDs; eine Warteschlange überträgt sie später ohne Duplikate. Der Server prüft dabei die aktuellen Rechte. Abgewiesene Änderungen bleiben als lokale Entwürfe erkennbar. Gleiche Namen werden nicht automatisch zusammengeführt. Jeder bekommt nur den zulässigen Ausschnitt des Stamms – einschließlich Suche und Zählern.

## 4. Kanzleifunktion, Mandatszuständigkeit und Handlung trennen

Ein IT-Admin bekommt dadurch keinen allgemeinen Dokumentzugriff; ein Partner ebenfalls nicht automatisch Zugriff auf sämtliche Mandate. Bearbeiten, Teamverwaltung, Teilen, Offline-Kopien, Finanzzugriff und das Aufheben fremder Reservierungen sind getrennte Rechte. Billing kann notwendige Kennungen, Leistungen und Rechnungen sehen, ohne Strategieunterlagen oder Chats lesen zu dürfen. Ausdrückliche Sperren und DMS-Rechte haben Vorrang. Dieselben Grenzen gelten für Suche, Vorschauen, Exporte, Agenten und abgeleitetes Wissen.

## 5. Klare Bedeutung von Speichern

Die automatische Wiederherstellungssicherung schützt den Editorentwurf separat. In einem geteilten nativen Projekt schreibt normales Speichern beziehungsweise aktiviertes Datei-Autosave in die Projektdatei; anschließend wird automatisch synchronisiert. Dafür ist kein zusätzlicher Veröffentlichungsklick nötig. „Lokal gespeichert, Übertragung ausstehend“ bedeutet dort nicht „privat“.

Bei DMS-Dokumenten wird zunächst die lokale Arbeitskopie gespeichert; erst bewusstes Zurückspeichern erzeugt die neue DMS-Fassung. Fachliche Freigaben oder das Versenden an Mandanten bleiben davon getrennt. Bei rein lokalen Projekten gibt es ohne Freigabe keinen Team-Sync.

## 6. Offene Tabs blockieren nichts

Eigene Fenster erwerben Schreibzuständigkeit erst beim tatsächlichen Bearbeiten. Ein sauberes, inaktives Fenster gibt sie ohne Übernahmedialog weiter; vorhandene Änderungen werden sicher übergeben. Zwischen Personen oder eigenen weiteren Geräten gibt es standardmäßig einen Bearbeitungshinweis und die bewusste Möglichkeit einer parallelen Fassung.

Aktivitätsmeldungen verfallen – vorgeschlagener Startwert: Meldung alle 20 Sekunden, nach 60 Sekunden nicht mehr aktuell. Fehlende Aktivität bedeutet niemals sicher „niemand bearbeitet“. Ein Dirty-Zustand wird nur angezeigt, wenn der betreffende Editor ihn tatsächlich kennt. Harte Reservierungen bleiben optional oder werden vom DMS verlangt; weitere verpflichtende Kanzleirichtlinien sind noch festzulegen.

## 7. Neue Fassungen ersetzen keine geöffnete Ansicht

Auch ohne eigene Änderungen erscheint „Neue Version verfügbar“, mit Vergleich und bewusster Übernahme. Bei eigenen Änderungen bleiben Entwurf und ursprüngliche Basis erhalten. Hintergrunddownload ist erlaubt, stilles Austauschen des Editorinhalts nicht. Agenten unterliegen denselben Schreibregeln; bei aktiver menschlicher Bearbeitung erzeugen sie standardmäßig eine Vorschlagsfassung. Eigene weitere Geräte zählen ebenfalls als unabhängige Schreiber.

## 8. Dokumentidentität und Versionierung als eigener Ausbau

Dokumente erhalten stabile IDs unabhängig von Namen und Pfaden. Versionen halten Basis, Herkunft und Inhalt fest. Beim verbindlichen Speichern werden Rechte, Reservierung und erwartete Basis gemeinsam geprüft. Verlorene Bestätigungen dürfen bei Wiederholung keine zweite Veröffentlichung erzeugen. Konflikte erhalten beide Fassungen; Wiederherstellen erzeugt eine neue Version.

Umbenennen, Verschieben und Löschen brauchen eigene Regeln. Gleicher Dateiinhalt allein beweist keine Dokumentidentität. Ein führendes DMS behält seine Dokument-/Versionsidentitäten und persönlichen Rechte; eine DMS-Arbeitskopie wird nicht konkurrierend über den nativen Team-Sync veröffentlicht. Einfache Speicheradapter werden nicht als vollwertige DMS-Integration dargestellt.

## 9. Offline ohne Vorbereitungsritual, mit ehrlichen Grenzen

Bereits vollständig lokal vorhandene, erlaubte Inhalte bleiben nutzbar. „Offline behalten“ ist eine zusätzliche Download-/Vorhalteoption. Fehlende Dateien und Cloud-KI werden dadurch nicht offline verfügbar.

Auch die Konfliktsicherung hat Grenzen: Beliebige externe Programme können eine frei zugängliche Datei überschreiben, bevor Legal Work das bemerkt. Die starke Verlustschutz-Zusage gilt deshalb für kontrollierte Schreibwege und integrierte Arbeitskopien, nicht pauschal für jeden Dateisystemzugriff.

## 10. Rechteentzug und Abrechnung als eigene Abläufe

Ein getrenntes Gerät lässt sich nicht sofort zurückrufen; exportierte Dateien ebenfalls nicht. Ungesendete Kanzleiarbeit soll über einen begrenzten Annahmekanal in Quarantäne gelangen, ohne Leserechte wiederherzustellen. Fehlt eine authentifizierbare Übergabe, braucht es einen administrativen Ersatzweg. Ein verschlüsselter, stärker kontrollierter Cache ist eine spätere Option.

Zeiterfassung hängt am Mandat, bei interner Arbeit am Projekt. Ablauf: Entwurf → eingereicht → freigegeben → abgerechnet → nachvollziehbare Korrektur. Offline-Sync darf keine Doppelbuchung erzeugen. Leistungen behalten ihren ursprünglichen Mandatsbezug beim Umhängen eines Projekts. Rechnungsempfänger und Billing Partner sind getrennte Referenzen; Agentenlaufzeit ist nicht automatisch abrechenbare Anwaltszeit.

## Umsetzung

| Etappe | Inhalt |
|---|---|
| 1 | Mandantenstamm, zunächst unauffällige Mandatsidentität, Projektzuordnung, Sidebar und Offline-Anlage. Bestehende Projektfreigaben bleiben maßgeblich. |
| 2 | Kanzleibesitz, übertragbare Verantwortung, Mandatsteams, Finanzrechte und kontrollierter Rechteentzug. |
| 3A | Editorwiederherstellung, eigene Fenster, Hinweise auf neue Fassungen und Agentenvorschläge. Kann parallel beginnen. |
| 3B | Native Dokument-IDs, Versionen, sichere Veröffentlichungen und Migration bestehender Projekte/Clients. |
| 3C | Ein vollständig geprüfter DMS-Pfad mit persönlicher Anmeldung und Zurückspeichern. |
| 4 / 5 | Zeiterfassung und Finanzintegration; später Co-authoring, Portale und Cloud-Aufträge. Finanzfunktionen müssen nicht auf sämtliche DMS-Arbeiten warten. |

## Verbleibende Entscheidungen

Offen sind verpflichtende Reservierungen für bestimmte Fälle, das zuerst unterstützte DMS und externe Editoren, Versionsdarstellung und Aufbewahrungsfristen, die sichtbare Mandatsübersicht sowie der Bedarf an streng verwalteten Offline-Kopien.

Pro Etappe enthält der [vollständige Plan](../../proposal/client-matter-collaboration.md) konkrete Abnahmeszenarien – insbesondere für mehrere Fenster/Geräte, Verbindungsabbrüche, konkurrierende Änderungen, Umbenennungen und Rechteentzug. Die konkreten Plattform- und Anbieterfähigkeiten sind vor der jeweiligen Implementierung zu prüfen.
