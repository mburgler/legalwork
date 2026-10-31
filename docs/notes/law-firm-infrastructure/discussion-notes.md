# Diskussion und Entscheidungen

**Zusammengestellt:** 31. Oktober 2026, aus der Diskussion und dem Planstand vom 23. Oktober 2026.

## Ausgangsfrage

Legal Work sollte anhand der jüngsten Pull Requests und Branches darauf untersucht werden, wie lokale Arbeit, Cloud-Ausführung, Dateisync, Berechtigungen, Dokumentreservierungen und Zusammenarbeit heute zusammenpassen. Zusätzlich sollte geprüft werden, welche Ansätze aus den eigenen Prototypen Pandect / Legal IDE und Project Temple sowie aus iManage, ADVOKAT, Clio und Actaport für eine Kanzleistruktur geeignet sind.

Das Produktproblem: Projekte allein bilden weder einen Mandanten mit mehreren Aufträgen noch unterschiedliche Mandatsteams und Finanzzuständigkeiten ausreichend ab. Die Struktur soll spätere Zeiterfassung und Kanzleifunktionen tragen, ohne die lokale Arbeitsweise aufzugeben. Die zunächst unklare Bezeichnung „ALD“ wurde als ADVOKAT präzisiert.

## Anforderungen aus der Diskussion

1. **Mehrere Projekte pro Mandant.** Ein Kunde darf nicht auf ein einziges Projekt reduziert werden. Mandant, Mandat und Projekt sind zu unterscheiden; die Oberfläche soll diese Trennung nicht durch zusätzliche Pflichtmasken aufdrängen.
2. **Interne Projekte bleiben möglich.** Recherche, Kanzleiorganisation und ähnliche Arbeit brauchen keinen Mandanten. Das Zuordnungsfeld bleibt leer; weder Benutzername noch „Intern“ wird als künstlicher Mandant angelegt. Freigaben und Kanzleizugehörigkeit sind davon unabhängig.
3. **Sidebar nach persönlicher Arbeitsweise.** Projektliste oder Gruppierung nach Mandanten beziehungsweise Mandaten. Der Ansichtswechsel erzeugt keine Datenkopien und verändert keine Rechte.
4. **Offline ohne vorgeschriebenes Vorbereitungsritual.** Bereits vorhandene Inhalte sollen einfach weiter verwendbar sein. „Offline behalten“ ist eine zusätzliche Verfügbarkeitsfunktion für noch zu ladende oder dauerhaft vorzuhaltende Inhalte.
5. **Viele Workspaces und Tabs dürfen keine Dauersperren erzeugen.** Öffnen und Lesen allein reservieren nichts. Der ursprüngliche Checkout-Gedanke sollte besonders Konflikte zwischen eigenen Fenstern verhindern.
6. **Änderungen anderer werden nicht still in eine geöffnete Ansicht übernommen.** Stattdessen Hinweis auf neue Versionen und bewusste Übernahme, auch bei einer sauberen geöffneten Ansicht. Eigene Änderungen bleiben erhalten.
7. **Bearbeitung und ungespeicherte Änderungen sichtbar machen.** Eine Warnung oder gezielte Reservierung kann sinnvoll sein. Der Nutzer hat keine pauschale harte Sperre für jedes geänderte Dokument verlangt; störende Locks sind ausdrücklich zu vermeiden.
8. **Kanzleifunktion ist nicht gleich Inhaltsberechtigung.** Partner, Bearbeiter, Assistenz, Billing-Mitarbeiter, Billing Partner, Finance Admin und IT-Admin brauchen unterschiedliche Zuständigkeiten und Rechte. Ein gemeinsamer Mandant eröffnet nicht automatisch sämtliche seiner Mandate.
9. **Bestehendes DMS respektieren.** Eine iManage-Anbindung soll fachliche Akten- und Dokumentidentitäten sowie persönliche Zugriffsgrenzen erhalten. Sie darf keine unkontrollierte zweite Dokumentablage mit weiteren Freigaben erzeugen.
10. **Zeiterfassung vorbereiten.** Leistungen, Rechnungsempfänger, Freigaben und Finanzsicht sollen später auf stabilen Mandatsbezügen aufbauen; interne Arbeit bleibt ebenfalls erfassbar.

## Rechercheergebnis in knapper Form

- Legal Work besitzt bereits Organisationen, gemeinsame Projekte und bidirektionalen Dateisync. Die fachliche Schicht für Mandanten, Mandate, Beteiligte, Mandatsteams und Finanzrechte fehlt im untersuchten Stand.
- Lokale Dateien, lokale Agentenausführung und lokale Modellinferenz sind unterschiedliche Eigenschaften. Ein lokales Projekt kann weiterhin Cloud-KI nutzen.
- Team-Dateisync, externe Speicheranbindungen und der damalige offene persönliche Cloud-Assistent sind getrennte Mechanismen. „Cloud“ ist keine einzelne Betriebsart.
- Die lokale Fensterkoordination schützt nicht automatisch fremde Geräte oder externe Programme. Die generische Route mit dem Namen „checkout“ lädt im untersuchten Stand eine Arbeitskopie; daraus folgt kein DMS-Lock.
- Pandect/Legal IDE und der spätere Project-Temple-Stand liefern sinnvolle Ansätze für Organisations-/Fallrollen, stabile Dateiidentitäten und Versionsherkunft. Freitext-Mandanten, pauschale Admin-Durchgriffe und unvollständige Konfliktabsicherung sollen nicht übernommen werden.
- iManage liefert das Vorbild für Dokument- und Aktenidentität, persönliche Rechte und DMS-Zuständigkeit; ADVOKAT für Akten-, Leistungs- und Kanzleiprozesse. Clio und Actaport zeigen, weshalb Finanzrechte getrennt von Dokumentzugriff geführt werden sollten.

Die Belege, Grenzen und geprüften Commit-IDs stehen im [vollständigen Bericht](Bericht.md). Diese Zusammenfassung aktualisiert die damaligen Befunde nicht auf den 31. Oktober.

## Adversarial Review und Auswahl der Empfehlungen

Auf ausdrücklichen Wunsch wurde Claude Code mit Fable 5.1 gestartet. Es prüfte eine eingefrorene Planfassung und ausgewählte lokale Quelldateien. Ergebnis: tragfähiges Produktkonzept, aber noch offene Architekturentscheidungen; neun priorisierte Findings. Der vollständige [Review](Fable-Adversarial-Review.md) bleibt separat erhalten.

| Thema | Übernommene Entscheidung |
|---|---|
| Mandatsidentität | Bereits im ersten Datenmodell anlegen. Das Projekt verweist optional auf ein Mandat; Mandanten ergeben sich aus dessen Beteiligten. Im einfachen Fall entsteht das Mandat ohne zusätzliche Pflichtmaske. |
| Speichern und Teilen | Wiederherstellungssicherung, Speichern der nativen Projektdatei und DMS-Zurückspeichern auseinanderhalten. Speichern im geteilten Ordner stellt zur automatischen Übertragung bereit. Keine private Entwurfszusage für diese Datei. |
| Aufwand für Versionierung | Als eigenen Umbau von Plattformvertrag, Dokumentregister, Sync, Editorbasis und Migration planen. Die erste Mandantenübersicht wartet nicht darauf. |
| Mandantenstamm | Organisationsdaten mit zulässigem lokalem Ausschnitt, stabilen IDs und Ausgangswarteschlange. Zuordnung unabhängig vom optionalen Projektdetails-Sync. |
| Aktivitätsanzeige | Neuer zugriffsgeschützter Kanal mit Person, Gerät/Sitzung, Ablauf und erkennbarer Unsicherheit. Kein Rückschluss „niemand bearbeitet“ aus fehlenden Meldungen. |
| Agenten und weitere Geräte | Als eigene Schreiber mit derselben Basis-/Rechteprüfung behandeln. Agenten erzeugen bei aktiver menschlicher Arbeit standardmäßig Vorschlagsfassungen. |
| Rechteentzug | Begrenzter Annahmekanal für Quarantänefassungen ohne Wiedererteilung von Leserechten; administrativer Ersatzweg bei fehlender Authentifizierung. |
| Verschlüsselter Cache | Späterer eigener Ausbau statt Voraussetzung für die Mandantenübersicht; externe Bearbeitung und Wiederherstellung gesondert nachweisen. |
| Eigene Fenster | Schreibzuständigkeit erst bei tatsächlicher Bearbeitung. Saubere, inaktive Ansichten erzwingen keinen Übernahmedialog; vorhandene Entwürfe werden sicher übergeben. |

Nicht übernommen wurden dauerhaft pfadbasierte native Dokumente als Endzustand, die sichere Erkennung von Umbenennungen allein durch Inhalts-Hashes und die Behauptung, ein geschützter Cache sei grundsätzlich unvereinbar mit automatischer Offline-Arbeit. Auch die pauschale Aufwandsschätzung „zwei Tabellen und ein Projektattribut“ wurde nicht übernommen, weil der tatsächliche Backendvertrag nicht geprüft werden konnte.

Die Abweichung einer vorgeschlagenen Funktion vom heutigen Code ist Implementierungsbedarf; sie ist nicht allein ein Widerspruch im Zielmodell.

## Stand nach der Überarbeitung

Die [Proposal-Datei](../../proposal/client-matter-collaboration.md) enthält die konsolidierte Richtung, konkrete Etappen und Abnahmeszenarien. Die vollständige Begründung und die Behandlung jedes Findings stehen im [Bericht](Bericht.md), insbesondere in den Abschnitten 5–11. Eine [kompakte Zusammenfassung](summary.md) macht die wesentlichen Inhalte ohne den Rechercheapparat zugänglich.

Offen bleiben die Reservierungsrichtlinien für bestimmte Fälle, die erste vollständige DMS-/Editorintegration, Versionsdarstellung und Aufbewahrung, die sichtbare Mandatsübersicht sowie der Bedarf an streng verwalteten Offline-Daten. Diese Entscheidungen blockieren nicht die erste Mandantenübersicht; konkrete Plattformverträge und Migrationen sind vor der betroffenen Implementierung auszuarbeiten.
