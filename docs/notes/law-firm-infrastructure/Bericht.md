# Legal Work: Kanzleistruktur, Berechtigungen und Zusammenarbeit

**Konzeptstand: 23. Oktober 2026, nach adversarial Review.** Die Empfehlungen berücksichtigen das Nutzerfeedback und den [unabhängigen Review durch Claude Code / Fable 5.1](Fable-Adversarial-Review.md). Übernommene und abgelehnte Empfehlungen sind in Abschnitt 11 dokumentiert. Abschnitte 2–4 beschreiben den ursprünglichen Recherche-Snapshot; Abschnitte 5–10 beschreiben den überarbeiteten Ziel- und Umsetzungsplan, keine bereits implementierten Funktionen. Die aktuelle Überarbeitung umfasst das Konzept, keine Änderungen am Produktcode. Ein erneuter vollständiger GitHub-/Herstellerabgleich wurde nicht durchgeführt.

**Ergebnis:** Legal Work hat bereits eine Organisationsebene, gemeinsame Projekte und bidirektionalen Dateisync. Was fehlt, ist eine verbindliche fachliche Schicht für **Mandanten, Mandate, Beteiligte, Mandatsteams und Finanzrechte**. Diese sollte auf den vorhandenen Grundlagen aufbauen. Ein zusätzliches Feld „Mandant“ oder ein Ordner oberhalb der Projekte würde das Problem nur optisch lösen.

Die empfohlene Richtung ist: **lokal arbeitsfähig, zentral berechtigt, dokumentbezogen versioniert und mit klarer Zuständigkeit des führenden Systems**. Bei einer iManage-Anbindung bleibt iManage für seine Dokumente und deren Zugriffsgrenzen führend; Legal Work ergänzt Arbeitsabläufe, lokale Entwürfe und KI-Unterstützung.

Die erste Etappe liefert Mandanten und eine einfache Projektzuordnung, mit einer zunächst unauffälligen Mandatsidentität im Hintergrund. Gespeicherte native Projektdateien werden weiter automatisch synchronisiert; DMS-Arbeitskopien werden bewusst zurückgespeichert. Neue Versionen ersetzen keine bereits geöffnete Ansicht. Verlässliche Dokumentidentitäten und Versionierung folgen als eigener Umbau, dessen Aufwand die Mandantenübersicht nicht blockiert.

## 1. Grundlage und Aussagegrenzen

Geprüft wurden die Metadaten der **200 zuletzt aktualisierten Pull Requests** von `eigenweltlabs/legalwork`, das Inventar von **81 upstream Branches**, die einschlägigen PR-Beschreibungen, Architekturdateien und zentrale Implementierungen. Der geprüfte GitHub-Entwicklungsstand ist [`bb77ce7`](https://github.com/eigenweltlabs/legalwork/commit/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d). Der lokale Checkout enthält diesen Stand plus Änderungen am Dashboard-Dateiexplorer; die hier maßgeblichen Sync- und Berechtigungsdateien entsprechen dem geprüften Entwicklungsstand.

Zusätzlich geprüft:

- Das private GitHub-Repo [`mburgler/legal_ide`](https://github.com/mburgler/legal_ide), Stand `5e9a978`, zuletzt gepusht am 25. Januar 2026. Das README nennt das Projekt **Pandect**.
- Den späteren privaten Nachfolger [`mburgler/project-temple`](https://github.com/mburgler/project-temple), Stand `d5231d9`, zuletzt gepusht am 18. Februar 2026. Er enthält das weiterentwickelte Rollen- und Dokumentversionsmodell.
- Die lokale Weiterentwicklung unter `/Users/mburgler/Coding/legal-ide`, deren letzter Commit vom 22. Mai 2026 stammt und für die kein Git-Remote eingetragen ist. Die lokale Version und der jüngste GitHub-Stand sind somit nicht identisch.
- Herstellerdokumentation von iManage, ADVOKAT, Clio und Actaport. „ALD“ wurde vom Nutzer als Advokat korrigiert.

**Dies ist eine Architektur- und Produktanalyse, keine Live-Abnahme.** „Gemergt“ bedeutet in den Entwicklungszweig aufgenommen, nicht automatisch im verwendeten Desktop-Installer oder Backend produktiv. Das begleitende private Repo `eigenweltlabs/model-api` war über die GitHub-Verbindung nicht zugänglich (404). Seine serverseitige Durchsetzung und sein Deployment konnte ich deshalb nicht unabhängig prüfen. Aussagen dazu sind als Vertrag bzw. dokumentiertes Verhalten des Legal-Work-Clients zu lesen. Es wurden keine Produktivkonfigurationen verändert und keine Tests neu ausgeführt; Testergebnisse in PRs sind Angaben der jeweiligen Autoren.

Die GitHub-Antworten der ursprünglichen Recherche enthalten teilweise Zeitstempel vom 22. Oktober, während das damals vorgegebene Clientdatum 21. Oktober 2026 war. Deshalb ist der Abruf über Commit-IDs und den beigefügten [GitHub-Snapshot](github-snapshot.json) eindeutig fixiert; er wird nicht als zeitlich exakter Produktivstand zum 21. Oktober ausgegeben.

## 2. Was in Legal Work bereits existiert

### 2.1 Vier unterschiedliche Bedeutungen von „Cloud“

| Mechanismus | Wo liegen Daten und wo läuft die Arbeit? | Befund |
|---|---|---|
| Lokales Projekt | Normale Dateien auf dem Rechner; lokale App und Legal-Work-Server/Agentenlaufzeit | Bestehende Grundlage. Ein lokaler Agent kann trotzdem einen externen Modellanbieter aufrufen. |
| Gemeinsamer Projekt-Sync | Lokale Projektkopien plus gemeinsame Kopie im Eigenwelt-Speicher | Seit PR #188 gemergt. Freigegebene Daten werden zwischen Rechnern synchronisiert. |
| Angebundener Speicher | Dateien bleiben etwa in Google Drive, SharePoint, SMB oder anderem Speicher; ausgewählte Arbeitskopien lokal | Bestehende Speicheradapter. Teamfreigabe einer Verbindung ist nicht gleich vollständiger Offline-Sync ihres Inhalts. |
| Persönlicher Cloud-Assistent | Zusätzliche persönliche Cloud-Replik und Cloud-Ausführung für mobile/Kanal-Aufträge | PR #232 offen. Der separate lokale Assistent in #312 ist ebenfalls offen. |

**Lokal gespeichert, lokal ausgeführt und lokal inferiert sind drei verschiedene Eigenschaften.** Ein Projekt kann vollständig lokal liegen, während ein KI-Auftrag Inhalte an einen Cloud-Modellanbieter sendet. Für den Flugzeugfall funktionieren nur tatsächlich lokale Dateien und Funktionen; ein Cloud-Modell, ein noch nicht heruntergeladenes Dokument oder ein Netzwerk-Connector wird durch Dateisync nicht offline nutzbar. [Projekt-Sync](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/docs/project-sync.md), [Cloud-Assistent #232](https://github.com/eigenweltlabs/legalwork/pull/232).

### 2.2 Die entscheidenden PRs und Branches

| PR / Branch | Status im Abruf | Bedeutung |
|---|---|---|
| [#131](https://github.com/eigenweltlabs/legalwork/pull/131), [#132](https://github.com/eigenweltlabs/legalwork/pull/132) | Gemergt | Speicherintegrationen und gemeinsam verwaltete Speicherverbindungen. |
| [#154](https://github.com/eigenweltlabs/legalwork/pull/154), [#188](https://github.com/eigenweltlabs/legalwork/pull/188) | Gemergt | Lokale Aufgaben mit Kanzlei-Sync; anschließend gemeinsame Projekte/Akten mit Dateisync. |
| [#185](https://github.com/eigenweltlabs/legalwork/pull/185), [#197](https://github.com/eigenweltlabs/legalwork/pull/197) | Gemergt | Projektarbeitsbereiche, externe Ordnerreferenzen, Projektkalender und Fristenfunktionen. |
| [#209](https://github.com/eigenweltlabs/legalwork/pull/209) | Gemergt | Kanzleirichtlinien, firmeneigene KI-Anbieter und Knowledge Hub. |
| [#213](https://github.com/eigenweltlabs/legalwork/pull/213) | Gemergt | Fensterübergreifende Bearbeitungsverantwortung, DOCX-Autosave und gemeinsame Chat-Warteschlange innerhalb einer Laufzeit. |
| [#227](https://github.com/eigenweltlabs/legalwork/pull/227) | Gemergt | Optionales Agenten-Sandboxing und Kanzleivorgaben. |
| [#268](https://github.com/eigenweltlabs/legalwork/pull/268) | Gemergt | Editierbare Projektmetadaten-Vorlagen und Bereich „Shared Projects“; keine neue fachliche Mandantenstruktur. |
| [#272](https://github.com/eigenweltlabs/legalwork/pull/272), [#308](https://github.com/eigenweltlabs/legalwork/pull/308) | Gemergt | Native Google-/Microsoft-Anbindungen; Teamverbindungen mit individueller Anmeldung oder delegiertem gemeinsamen Konto. |
| [#309](https://github.com/eigenweltlabs/legalwork/pull/309), [#310](https://github.com/eigenweltlabs/legalwork/pull/310) | Gemergt | Schutz vor zusätzlichem Dokument-Sync in bereits vom Cloud-Anbieter synchronisierten lokalen Ordnern; robustere Erkennung. |
| [#274](https://github.com/eigenweltlabs/legalwork/pull/274), [#283](https://github.com/eigenweltlabs/legalwork/pull/283) | Gemergt | Gebündelte Integration u. a. von Wikis/Actaport sowie Wiki-Zugriffen und Dashboard-Lebenszyklus. Teil-PRs wurden teils ohne eigenen Merge geschlossen. |
| [#232](https://github.com/eigenweltlabs/legalwork/pull/232), `codex/legalwork-cloud-sync` | Offen; `aee752e` | Persönlicher Cloud-Assistent, private Replikation, mobile Ausführung, Checkpoints und Ausführungs-Leases. |
| [#312](https://github.com/eigenweltlabs/legalwork/pull/312), `codex/johannes-assistant` | Offen; `553bf94` | Lokaler persönlicher Assistent; Cloud-Aktivierung bleibt in #232. |
| [#303](https://github.com/eigenweltlabs/legalwork/pull/303), `codex/connector-manager` | Offen; `b236121` | Zentraler lokaler Besitzer von MCP-OAuth-Verbindungen; verhindert konkurrierende Token-Erneuerungen. Kein Mandatsrollenmodell. |

Ein noch existierender Branch beweist nicht, dass sein Inhalt unveröffentlicht ist: Einige Änderungen wurden über Sammel-PRs übernommen. Umgekehrt beweist ein PR-Text „ready“ keinen Merge. Die obige Unterscheidung beruht auf den GitHub-Statusfeldern und den dokumentierten Integrationswegen.

### 2.3 Wie der gemeinsame Projekt-Sync funktioniert

Ein Projekt bleibt zunächst lokal. Der Eigentümer kann einzelne Kanzleimitglieder oder die gesamte Kanzlei freischalten. Administratoren erhalten laut Projekt-Sync-Vertrag **keinen automatischen Projektzugriff**. Empfänger bekommen eine lokale Projektkopie. Es gibt Schalter für Dokumente, Notizen, Aufgaben, Aufnahmen, Projektdetails, Reviews und Kalender. Chats werden nicht automatisch mit dem Projekt synchronisiert.

Die Implementierung besitzt bereits nützliche Grundlagen: dauerhafte Projekt-IDs, eine lokale Ausgangswarteschlange, einen Drei-Wege-Vergleich zwischen lokalem Stand, gemeinsamem Stand und letzter gemeinsamer Basis, versionsgebundene Schreibvorgänge und Ereignisse, die neue Sync-Runden auslösen. Dateiübertragungen laufen in 8-MiB-Blöcken über signierte Speicherlinks; Prüfsummen sichern den Transfer.

Konflikte sind nach Inhaltstyp unterschiedlich behandelt:

- Markdown-/Textänderungen werden nach Möglichkeit zeilen- und wortweise zusammengeführt.
- Bei kollidierenden Änderungen bzw. anderen Dateiformaten wie DOCX/PDF bleiben beide Fassungen erhalten. Die gemeinsame Version behält den Namen; die lokale Fassung erhält einen Konfliktnamen. Eine Person entscheidet über das Ergebnis.
- Eine Änderung gewinnt gegen eine konkurrierende Löschung. Entfernte Dateien landen lokal zunächst im Sync-Papierkorb, regulär 30 Tage lang.
- Fehlende Ordner pausieren den Sync. Größere Löschwellen werden zur Entscheidung angehalten.
- Reviews besitzen eigene Zusammenführungsregeln und eine zeitlich begrenzte Anzeige der laufenden Bearbeitung auf einem anderen Rechner.

Das ist eine brauchbare Basis für asynchrone Zusammenarbeit. Es ist noch kein gleichzeitiges gemeinsames Bearbeiten eines Word-Dokuments. Der Textmerge versteht außerdem keine juristische Bedeutung: Zwei technisch konfliktfreie Änderungen können inhaltlich widersprüchlich sein. Für freizugebende Schriftsätze oder Verträge bleibt eine fachliche Prüfung nötig.

Aktuelle Grenzen: Alle für ein Mitglied sichtbaren Sync-Projekte werden lokal repliziert; es gibt im dokumentierten Team-Sync keinen selektiven Download nach Bedarf. Leere Ordner und Dateien über 256 MiB werden nicht synchronisiert. Eine Übernahme eines Projekts, dessen Eigentümer die Kanzlei verlassen hat, ist in der Dokumentation ausdrücklich noch offen. [Implementierung und Grenzen](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/docs/project-sync.md).

### 2.4 Welche Rechte heute voneinander getrennt existieren

| Ebene | Vorhandenes Modell | Was daraus noch nicht folgt |
|---|---|---|
| Kanzleikonto | Im sichtbaren Vertrag `org:admin` / `org:member`, Nutzungsbudgets und Einstellungen | Keine Rollen für Honorarabrechnung einzelner Mandate. Die vorhandene „Billing“-Logik betrifft wesentlich das Software-/KI-Konto. |
| Geteiltes Projekt | `owner` / `member`; benannte Mitglieder oder gesamte Kanzlei | Kein durchgängiges Viewer-/Editor-/Mandatsleiter-/Billing-Modell. Mitglieder bearbeiten grundsätzlich den geteilten Inhalt; der Owner verwaltet Zugriff und Umfang. |
| Verbindung zum Legal-Work-Server | Token-Scopes `owner`, `collaborator`, `viewer` | Diese technischen Verbindungsrechte sind nicht identisch mit der Mitgliedschaft eines Kanzleimandats. |
| Speicher | Provider-Berechtigung, Read-only-Einstellung und lokale Schreibfreigaben | Ein freigegebenes Projekt verschafft nicht automatisch Zugriff auf einen externen Speicher. |
| KI-Werkzeuge | Allow/Ask/Deny, Sandbox, zugelassene Verzeichnisse, Kanzleirichtlinien | Eine Bestätigung einer KI-Aktion ersetzt keine fachliche Berechtigung auf Mandatsdaten. |
| Wiki-/Agentenzugriff | Begrenzte projektbezogene Lesefreigaben und weitere Schutzmechanismen | Noch keine allgemeine, einheitliche Vererbung von Mandatsrechten auf sämtliche Daten und Ableitungen. |

Quellen: [Projektvertrag](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/apps/server/src/eigenwelt-projects.ts), [Server-Scopes](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/apps/server/src/types.ts), [Kanzleirichtlinien](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/packages/types/src/org-policy.ts), [Konto-/Nutzungsrechte](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/packages/types/src/usage-control.ts).

### 2.5 Check-out, Check-in und Dokumentversionen

Hier sind drei Mechanismen auseinanderzuhalten:

1. **Fenstersperre:** Legal Work koordiniert mit Web Locks und BroadcastChannel, welche Ansicht innerhalb desselben Browserprofils ein Original bearbeiten darf. Das schützt mehrere eigene Fenster. Es sperrt nicht automatisch den Rechner einer Kollegin, externe Word-Instanzen oder einen anderen Server.
2. **Versionsprüfung beim Speichern:** Externe Speicher werden mit ETag, Generation oder Inhalts-Hash geprüft. Veraltete Schreibvorgänge werden soweit vom Backend unterstützt abgewiesen. Bei SMB/SFTP/FTP kann zwischen letzter Hashprüfung und Ersetzung trotzdem ein externer Schreibvorgang stattfinden; die Dokumentation benennt diese Grenze.
3. **DMS-Auschecken:** Dafür braucht es eine serverseitig anerkannte Reservierung eines konkreten Dokuments sowie einen kontrollierten Check-in. Die Route namens `/checkout` in Legal Work lädt gegenwärtig eine Arbeitskopie herunter. Der allgemeine `StorageAdapter` hat keine Check-out-/Check-in-/Lock-Operationen. Aus diesem Routennamen darf keine kanzleiweite Sperre abgeleitet werden.

Revisionsprüfungen, Wiederherstellungsentwürfe, Wiki-Historie und Review-Snapshots sind vorhanden. Ein einheitlicher DMS-Lebenszyklus mit unveränderlichen Versionen für sämtliche Projektdokumente ist damit nicht nachgewiesen. [Fensterkoordination](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/apps/app/src/react-app/domains/session/artifacts/document-ownership.ts), [Checkout-Route](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/apps/server/src/routes/file-storage.ts#L431), [Speichervertrag](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/apps/server/src/file-storage/common.ts), [Speichergrenzen](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/docs/file-storage.md).

### 2.6 Zwei besonders wichtige Folgen für Kanzleien

**Entzug der Freigabe ist heute kein vollständiger Rückruf aller Kopien.** Sync-erzeugte Empfängerkopien werden grundsätzlich entfernt. Bei ungesendeten Änderungen darf der Benutzer die Kopie aber als eigenes lokales Projekt behalten; selbst eingebrachte Ordner bleiben ebenfalls erhalten. Heruntergeladene Kopien externer Speicher bleiben lokal. Das ist nachvollziehbarer Schutz vor Datenverlust, genügt aber nicht als alleinige Umsetzung einer strengen Mandatssperre. [Entzug im Code](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/apps/server/src/project-sync.ts#L553).

**Ein Dokument sollte genau einen zuständigen Sync haben.** #309 verhindert das zusätzliche Hochladen gewöhnlicher Dokumente aus erkannten OneDrive-/ähnlichen Cloud-Ordnern über den Eigenwelt-Projekt-Sync. Zuvor konnte bereits das Hashen Online-only-Dateien herunterladen. Die Änderung migriert allerdings nicht global alle schon hochgeladenen Kopien oder Einstellungen anderer Rechner. Für ein DMS ist die richtige Konsequenz eine explizite Quellenzuordnung pro Dokument bzw. Dokumentbereich, statt allein auf lokale Ordnererkennung zu vertrauen. [#309](https://github.com/eigenweltlabs/legalwork/pull/309), [#310](https://github.com/eigenweltlabs/legalwork/pull/310).

### 2.7 Was der offene Cloud-Assistent zusätzlich macht

#232 verwendet den vorhandenen Projekttransport. Persönlicher Zustand liegt in einem nur dem Eigentümer zugänglichen Projekt mit besonderem Zweck; gemeinsame Dokumente werden nicht nochmals in diesen persönlichen Bereich kopiert. Ein Cloud-Prozess besitzt über eine Lease die Checkpoint-Verantwortung. Desktop-Anfragen laufen lokal, mobile Anfragen in der VM; importierte Nachrichten werden nicht erneut als lokale Aufträge abgespielt.

Die Replik umfasst auch persönliche Historie und sichere Laufzeiteinstellungen. Allgemeine Credential-Stores sind aus Checkpoints ausgeschlossen. Der spätere Connector-Zusatz führt native Nango-Referenzen über eine eigene Kontoregistrierung zu; Cloud-Schreibrechte starten unabhängig vom Desktop deaktiviert. Das ist eine separate Freigabe- und Bereitstellungsstrecke, kein Kopieren sämtlicher lokaler OAuth-Geheimnisse. Die PR nennt noch erforderliche Backend-/Guest-Änderungen und ausstehende Live-Abnahme.

**Für das Zielmodell muss private Cloud-Verfügbarkeit getrennt von Teamfreigabe und Kanzleierlaubnis geführt werden.** „Nicht mit Kollegen geteilt“ bedeutet auf diesem Branch nicht zwingend „nur auf meinem Gerät“. [Architektur des offenen Branches](https://github.com/eigenweltlabs/legalwork/blob/aee752e8a34c147286fb1c2d4acee4488ff5eb4d/docs/cloud-assistant-sync.md), [PR mit Connector-Nachtrag](https://github.com/eigenweltlabs/legalwork/pull/232).

## 3. Was an Pandect / Legal IDE / Project Temple gut war

Die spätere Fassung `project-temple` ist für diese Frage aussagekräftiger als das ältere `legal_ide`. Sie enthält tatsächlich implementierte fachliche Ansätze:

- **Organisation, Benutzer und Fall sind eigene Entitäten.** Der Fall hat eine stabile ID und gehört zur Organisation.
- **Globale Rolle und Fallmitgliedschaft sind getrennt.** `Superadmin/Admin/User` einerseits; `CaseUser` mit `owner/editor/viewer` andererseits.
- **Einladungsrecht ist separat.** `can_invite` muss nicht automatisch jeder Editor erhalten.
- **Sichtbarkeit ist ein eigener Zustand.** Private, organisationsweite und globale Sichtbarkeit sind modelliert.
- **Dokumentidentität und Ordnerstruktur sind getrennt.** Dateien besitzen IDs und Elternbeziehungen; ein Pfad ist nicht ihre einzige Identität.
- **Versionen besitzen Herkunft.** `FileVersion` enthält Versionsnummer, Benutzer-/KI-Herkunft, Bezeichnung und Wiederherstellungsbezug. Es existieren Routen zum Auflisten, Erzeugen, Herunterladen und Wiederherstellen.
- **Der fachliche Kern soll ohne KI funktionieren.** Das ist eine gute Voraussetzung für Offline-Arbeit und verlässliche Kanzleiprozesse.

Quellen: [Datenmodell](https://github.com/mburgler/project-temple/blob/d5231d9faa221e857f1266a374b7b27f8b5077dd/backend/app/models.py), [Fall- und Versionsrouten](https://github.com/mburgler/project-temple/blob/d5231d9faa221e857f1266a374b7b27f8b5077dd/backend/app/api/cases.py), [Projektbeschreibung](https://github.com/mburgler/project-temple/blob/d5231d9faa221e857f1266a374b7b27f8b5077dd/README.md).

**Übernehmen würde ich diese Trennung, nicht die Implementierung unverändert.** Vier Grenzen fallen konkret auf:

1. `client_name` und `opponent_name` sind Textfelder. Auch Temple hat kein echtes Mandantenregister, keine Mehrmandantenbeziehung und kein eigenständiges Honorar-/Zeitmodell.
2. Superadmin und teilweise Organisationsadmin umgehen die Fallprüfung. Für besonders vertrauliche Mandate wäre ein pauschaler Admin-Durchgriff die falsche Voreinstellung. Auch globale Sichtbarkeit sollte nicht Teil des normalen Mandatsdialogs sein.
3. Die Route zum Entfernen eines Mitglieds verlangt lediglich Bearbeitungszugriff. Sie trennt damit das Bearbeiten von Dokumenten nicht konsequent vom Verwalten des Teams. Das illustriert, warum ein gutes Rollenmodell allein noch keine lückenlose Durchsetzung ergibt.
4. Beim Inhaltsupload wird die Hauptdatei überschrieben; ein erwarteter Basisstand wird dort nicht verlangt. Versionserzeugung ist optional. Eine Historie schützt daher nicht automatisch gegen verlorene konkurrierende Änderungen. Ein verteilter Offline-Sync oder DMS-Check-out ist in den geprüften Kernpfaden nicht umgesetzt.

[Admin-/Fallprüfung](https://github.com/mburgler/project-temple/blob/d5231d9faa221e857f1266a374b7b27f8b5077dd/backend/app/api/cases.py#L36), [Mitglied entfernen](https://github.com/mburgler/project-temple/blob/d5231d9faa221e857f1266a374b7b27f8b5077dd/backend/app/api/cases.py#L1029), [Datei überschreiben](https://github.com/mburgler/project-temple/blob/d5231d9faa221e857f1266a374b7b27f8b5077dd/backend/app/api/cases.py#L741).

## 4. Was etablierte Systeme beitragen

### iManage: Dokumente und Zugriff als eigenständige Fachobjekte

iManage unterscheidet Clients, Matters/Workspaces, Dokumente und Versionen. Security Policy Manager ergänzt zugriffsbeschränkende Regeln auf Mandanten-/Mandatsebene, einschließlich Teamzuordnung, ausgeschlossener Personen und Konfliktkonstellationen. Das ist mehr als eine globale Hierarchie aus Partnern und Associates. [SPM-Begriffe](https://docs.imanage.com/cloud/spm-admin-help/en-US/Getting_started_with_iManage_Security_Policy_Manager.html), [Zusammenspiel der Sicherheitsregeln](https://docs.imanage.com/cc-help/10.4.0/en/Container_and_Document_Security.html).

iManage Drive erlaubt gezielte Offline-Verfügbarkeit von Dateien und Containern. Das Vorhandensein einer Offline-Kopie ist nicht gleichbedeutend mit exklusiver Bearbeitung: Es gibt eigene Konfliktbehandlung und Check-in-Optionen. Zusätzlich existiert gemeinsames Bearbeiten über Microsoft 365; ein exklusiv ausgechecktes Dokument ist für andere dabei nur lesbar. [Offline-Auswahl](https://docs.imanage.com/drive-windows-help/10.12.0/en/Making_content_available_for_offline_use.html), [Konfliktauflösung](https://docs.imanage.com/drive-windows-help/10.12.0/en/Conflict_resolution.html), [Co-authoring](https://docs.imanage.com/work-nextgen-coauthoring/en-US/FAQ__.html).

Legal Works iManage-Eintrag ist im geprüften Stand ein als Preview markierter Remote-MCP-Connector mit Tokenanmeldung. Das ist noch keine nachgewiesene Abbildung eines gesamten Mandats- und Dokumentlebenszyklus. Der aktuelle Hersteller-MCP bietet allerdings bereits sowohl Lese- als auch Schreibwerkzeuge, darunter Uploads und Workspace-Änderungen. Ihn pauschal als read-only zu bezeichnen wäre falsch. Ein eigener Check-out-/Check-in-Vertrag ist in der eingesehenen Werkzeugliste nicht ausgewiesen. Für eine vollständige Integration müssen die benötigten Fähigkeiten je System und Tenant geprüft und gegebenenfalls über zusätzliche APIs oder die native Anwendung bedient werden. [Legal-Work-Katalog](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/apps/app/src/app/constants.ts#L200), [iManage-MCP-Werkzeuge](https://docs.imanage.com/ai/mcpserver-user-help/en-US/Tools_for_iManage_Work_connector.html).

### ADVOKAT: Akte, Leistungen und Kanzleiprozesse gehören zusammen

ADVOKAT umfasst neben Aktmanagement u. a. Leistungserfassung, Honorarabrechnung, Fristen/Termine, ERV, Fremdgeld, Buchhaltung und Auswertungen. Die mobile Anwendung unterstützt mitgenommene Offline-Akten; die Timer-App kann Zeiten ohne Verbindung aufzeichnen und später übertragen. [Module](https://advokat.at/software/module/), [Mobile Anwendung](https://play.google.com/store/apps/details?hl=de_AT&id=at.advokat.android).

Die Rechte reichen bis zu konkreten Funktionen: Stundensätze verwalten, eigene Leistungen ändern, bereits abgerechnete Leistungen ändern oder Fristen löschen. Ein pauschales „darf die Akte bearbeiten“ genügt dafür nicht. Zugleich nennt der Hersteller Grenzen: App-Rechte schützen normale Dateien nicht automatisch gegen den Windows Explorer. Automatische Dateirechte hängen von der konkreten SharePoint-Anbindung ab. Auschecken und Versionen sind ebenfalls integrationsabhängig. [Funktionsrechte](https://advokat.atlassian.net/wiki/spaces/KB/pages/1631552727), [Security-FAQ](https://advokat.atlassian.net/wiki/spaces/KB/pages/45191209).

**Lehre für Legal Work:** ADVOKAT liefert das fachliche Vorbild für Leistungs- und Aktenprozesse; seine Dateisystemgrenzen sollte man bei einem neuen Offline-Konzept ausdrücklich besser lösen.

### Clio und Actaport: Finanzzugriff kann getrennt von Aktenzugriff sein

Clio trennt Standard- und benutzerdefinierte Rollen, u. a. General Access, Billing, Accounts und Reports. Abrechnungsrechte und Sichtbarkeit von Stundensätzen sind eigene Einstellungen. Actaport zeigt die Trennung besonders anschaulich: Eine beschränkte Akte kann unsichtbar bleiben, während Rechnungen und Stundenpositionen mit gesonderten Finanzrechten zugänglich sind. Die Eintragung als Sachbearbeiter oder Assistenz hebt eine Aktenbeschränkung nicht automatisch auf. [Clio-Rollen](https://help.clio.com/hc/en-us/articles/9200279456667-Roles-and-Permissions-in-Clio-Manage), [Billing-Rechte](https://help.clio.com/hc/en-us/articles/9285360193819-Permissions-and-Billing-Rates), [Actaport-Aktenzugriff](https://actaport.zendesk.com/hc/de/articles/4756389596572-Aktenzugriff-beschr%C3%A4nken).

Actaport ist zusätzlich ein pragmatischer Integrationskandidat, weil Legal Work bereits eine gebündelte MCP-Anbindung enthält. Eine angebundene Kanzleisoftware und eine bloße Dateifreigabe sollten im Produkt erkennbar verschiedene Fähigkeiten haben.

## 5. Empfohlenes Fachmodell

### 5.1 Nutzeransicht und Datenmodell

**Ein Mandant kann beliebig viele Projekte haben.** Ein Projekt benötigt keinen Mandanten: freie Recherche und Kanzleiorganisation bleiben eigenständige Projekte mit leerer Mandantenzuordnung. Weder der Benutzername noch „Intern“ wird als künstlicher Mandant gespeichert. „Intern“ kann eine Projektart oder eine Anzeigegruppe sein. Eine leere Mandantenzuordnung entscheidet nicht über Eigentum oder Zugriff: Auch ein internes Projekt kann der Kanzlei gehören und mit einem Team geteilt sein; ein noch nicht zugeordnetes Projekt ist nicht automatisch intern.

Die Sidebar erhält eine persönliche Ansichtswahl: **Projekte**, **nach Mandanten** und, soweit fachlich verwendet, **nach Mandaten**. Gruppierung und Sortierung ändern weder Identitäten noch Freigaben. Eine mögliche Ansicht „nach Mandanten“:

```text
Kanzlei
├── Muster GmbH
│   ├── Unternehmenskauf 2026
│   ├── Arbeitsrechtliche Beratung
│   └── Rechtsstreit Lieferant X
├── Weitere Mandanten …
└── Ohne Mandant
    ├── Freie Recherche
    └── Kanzleiorganisation
```

Begrifflich ist der Mandant die Person oder das Unternehmen, das Mandat der konkrete Auftrag/Fall und das Projekt der Arbeitskontext in Legal Work. **Das Mandat erhält bereits im ersten Ausbau eine stabile Identität.** Häufig deckt ein Projekt genau ein Mandat ab. Dafür verlangt die Oberfläche keine doppelte Anlage: Wird einem neuen Projekt ein Mandant zugeordnet, entsteht im Hintergrund ein neues Mandat oder es wird bewusst ein vorhandenes ausgewählt. Ein gleicher Mandantenname führt niemals automatisch dazu, dass zwei Projekte dasselbe Mandat teilen. Mehrere Projekte können zum selben Mandat gehören, etwa „Due Diligence“ und „Verhandlung“. Die zusätzliche Navigationsebene kann im einfachen Einprojektfall verborgen bleiben.

Eine Mandatsübersicht kann Beteiligte, Verantwortliche, Projekte, Dokumente, Aufgaben, Fristen und – soweit berechtigt – Budget und Leistungen bündeln. Technische Fenster, Tabs und wiederherstellbare Layouts sind davon unabhängig: Ein zusätzlich geöffneter Workspace erzeugt weder ein neues Mandat noch neue Rechte oder eine Dokumentreservierung.

| Objekt | Aufgabe |
|---|---|
| Kanzlei / Organisation | Sicherheits-, Verwaltungs- und Zuständigkeitsgrenze. |
| Mandant / Person / Unternehmen | Wiederverwendbare Identität, Kontakte, Kennungen, Beziehungen. |
| Mandat / Akte | Fachlicher Auftrag mit Aktenzeichen, Lebenszyklus, Team, Richtlinien und Finanzbezug. |
| Mandatsbeteiligter | Verknüpft eine Person/Organisation in einer Rolle: Mandant, Gegner, Gericht, Versicherer, Kontakt, Rechnungsempfänger. |
| Projekt | Arbeitskontext mit optionaler Mandatsreferenz. Seine Mandanten ergeben sich aus diesem Mandat; kein unabhängiges, widersprüchliches Mandantenfeld. Auch intern oder noch nicht zugeordnet möglich. |
| Fenster / Tabs / Workspace-Layout | Persönliche Ansicht auf bestehende Projekte und Dokumente; keine fachliche Eigentums- oder Berechtigungsgrenze. |
| Dokument und Dokumentversion | Dauerhafte Identität sowie nachvollziehbare Fassungen; unabhängig vom Dateinamen. |
| Teamzuordnung | Person/Gruppe, Zuständigkeit, Berechtigungsrolle und Gültigkeitszeitraum. |
| Leistung / Abrechnung | Fachliche Finanzobjekte mit eigener Berechtigung und Freigabestatus. |

Der sichtbare Baum ist nicht das vollständige Datenmodell. Ein Mandat kann mehrere Mandanten haben; eine Versicherung kann Rechnungsempfänger sein, ohne Mandant zu sein. Ein Konzern kann mehrere eigenständige Mandanten enthalten. Ein gemeinsamer Konzernknoten darf nicht automatisch alle Mandate für alle Teams öffnen.

Für den ersten Ausbau gelten diese Beziehungen:

- Ein Projekt gehört zu **höchstens einem Mandat**, ein Mandat kann mehrere Projekte enthalten.
- Ein Mandat kann **mehrere Mandanten** über Beteiligtenbeziehungen haben. Die erste Oberfläche kann mit einem Mandanten beginnen, das Datenmodell darf die Mehrfachbeziehung nicht ausschließen.
- Interne und noch nicht zugeordnete Projekte haben eine leere Mandatsreferenz und damit keine abgeleitete Mandantenzuordnung. Ihre Kanzleizugehörigkeit und Freigaben bestehen unabhängig davon.
- Die Sidebar darf dasselbe Projekt bei mehreren berechtigten Mandanten anzeigen; es bleiben Verweise auf dieselbe Projekt-ID, keine Kopien. Zähler berücksichtigen nur sichtbare, eindeutige Projekte.
- Zusammenführen von Mandanten und Umhängen von Projekten sind nachvollziehbare Verwaltungsaktionen. Sie ändern weder still die Freigaben noch rückwirkend den Mandatsbezug bereits erfasster Leistungen. Namen, Identitäten und lokale Dateipfade bleiben getrennt.

Damit ist die fachliche Identität früh festgelegt. Offen bleibt die Gestaltung der sichtbaren Mandatsübersicht, nicht die Existenz des Mandats im Datenmodell.

### 5.2 Drei getrennte Rollenfragen

**Kanzleifunktion:** Darf eine Person Benutzer, Integrationen, Finanzkonfiguration oder Sicherheitsregeln verwalten?

**Mandatszuständigkeit:** Ist sie Relationship Partner, verantwortlicher Partner, Billing Partner, Associate, Assistenz oder externer Mitarbeiter dieses Mandats?

**Konkretes Recht:** Darf sie Dokumente lesen, bearbeiten, herunterladen, teilen, Teammitglieder verwalten, eigene Zeiten erfassen, Leistungen freigeben oder Rechnungen versenden?

„Partner“ ist eine berufliche Funktion, keine automatische Leseberechtigung für jedes Mandat. „Mandant betreuen“ ist ebenfalls nicht dasselbe wie „alle Mandate dieses Mandanten sehen“.

Empfohlene Standardvorlagen:

| Rolle im jeweiligen Geltungsbereich | Dokumentinhalt | Team / Freigabe | Leistungen / Finanzen |
|---|---|---|---|
| IT-/Organisationsadmin | Kein automatischer Inhaltszugriff | Benutzer und technische Konfiguration; begrenzte Verwaltungsrechte | Kein automatischer Honorarzugriff |
| Mandatsverantwortlicher Partner | Freigegebene Mandatsinhalte | Team und zulässige Delegationen | Mandatsbudget und gesondert zugewiesene Freigaben |
| Bearbeiter / Associate | Lesen und bearbeiten | Standardmäßig kein Recht, Zugriff zu erweitern | Eigene Leistungen erfassen und einreichen |
| Assistenz | Nach Aufgabenprofil | Keine automatische Rechteverwaltung | Eigene/zugewiesene Erfassung, Fristenrechte separat |
| Billing-Mitarbeiter | Nur bei zusätzlicher Inhaltsfreigabe | Keine | Erforderliche Mandatskennungen, Leistungs- und Rechnungsbearbeitung |
| Billing Partner | Nur bei zusätzlicher Inhaltsfreigabe | Keine automatische | Freigabe definierter Mandatsabrechnungen und Korrekturen |
| Finance Admin | Kein pauschaler Dokumentzugriff | Finanzrollen und Konfiguration | Honorarsätze, Finanzprozesse und Auswertungen nach Scope |
| Externer / Mandant | Explizit freigegebener Ausschnitt | Keine Weiterfreigabe als Default | Keine internen Finanz-/Leistungsdaten |

Mehrere Rollen können einer Person gemeinsam zugewiesen werden. Die Rechte aus Rollen werden gesammelt, aber ausdrückliche Sperren, DMS-Beschränkungen und Informationsbarrieren haben Vorrang. Ein dokumentierter Notfallzugriff sollte eng befristet und protokolliert sein, kein unsichtbarer Superadmin-Bypass.

Das Recht, eine fremde Dokumentreservierung aufzuheben, ist eine eigene delegierbare Befugnis mit Begründung und Protokollierung. Sie ist kein allgemeines IT-Adminrecht. Eine Aufhebung verwirft keine Entwürfe und berechtigt den früheren Reservierungsinhaber nicht, später einen neueren Stand zu überschreiben.

### 5.3 Ein einheitlicher Entscheidungsweg

Für jeden Zugriff wird dieselbe Entscheidung benötigt – in der Oberfläche, im Sync, bei der Suche, im Export und im Agenten:

**Erlaubte Rollenaktion im passenden Scope + gültige Mitgliedschaft + erlaubte Daten-/Geräterichtlinie + Berechtigung des Quellsystems; keine entgegenstehende Sperre.**

Finanzrechte bekommen einen eigenen Datenumfang. Billing darf beispielsweise Aktennummer, Rechnungsempfänger, notwendige Leistungstexte und Beträge sehen, ohne dadurch Zugang zu Schriftsätzen oder Strategie-Chats zu erhalten. Besonders geschützte Leistungsbeschreibungen können eine nochmals engere Sicht erfordern.

Volltextindizes, Embeddings, Vorschauen, Exporte, Chat-Anhänge und KI-Zusammenfassungen brauchen dieselben Schutzbezüge wie ihre Quellen. Ein Kanzlei-Wiki darf nicht zum Umweg um eine Mandatssperre werden. Für wiederverwendbares Wissen braucht es eine bewusste Freigabe oder Bereinigung; bei mehreren Quellen gilt ohne solche Freigabe die restriktive Kombination ihrer Schutzregeln. Schon Trefferzahlen, Dateinamen und Benachrichtigungen können geschützte Informationen offenlegen.

### 5.4 Mandantenstamm, lokale Anlage und Sichtbarkeit

Mandanten, Mandate und ihre Beziehungen sind **Organisationsdaten mit eigener lokaler Replik und Ausgangswarteschlange**. Die Mandatszuordnung ist ein kontrolliertes Attribut des Projekts, unabhängig vom optionalen Sync-Schalter für freie Projektdetails. Eine lokale Kopie der Zuordnung hilft bei Wiederherstellung; sie ist keine zweite unabhängig bearbeitbare Wahrheit.

Für bereits berechtigte Kanzleimitglieder ist die Anlage auch offline möglich: lokal erzeugte IDs und wiederholbar übertragbare Operationen verhindern doppelte Datensätze nach Verbindungsabbrüchen. Abhängige Vorgänge werden in der Reihenfolge Mandant → Mandat/Beteiligte → Projektzuordnung übertragen. Vor Annahme prüft der Server die aktuellen Rechte. Wird eine Offline-Anlage abgewiesen, bleibt sie als gekennzeichneter lokaler Entwurf zur Klärung erhalten und erscheint nicht als erfolgreich geteiltes Kanzleimandat. Namensgleichheit erzeugt einen Vorschlag zur Zusammenführung, keinen automatischen Merge; konkurrierende Zuordnungsänderungen benötigen eine Versionsprüfung.

Die lokale Replik enthält nur den zulässigen Ausschnitt. Ein Organisationsstamm bedeutet nicht, dass jedes Mitglied sämtliche Mandanten, Mandate oder Beziehungen auflisten kann. Für die erste Sidebar werden nur die minimal erforderlichen Stammdaten zu sichtbaren Projekten geliefert; eine umfassende Mandantenverwaltung erhält eigene Rechte. Die Auswahl, Suche, Zähler und Offline-Replik dürfen keine verborgenen Mandate verraten.

Im ersten Ausbau bleiben bestehende Projektfreigaben maßgeblich. Das gemeinsame Mandat verbindet Projekte fachlich, ohne deren Teams automatisch zu vereinigen. Eine spätere Vererbung von Mandatsrechten auf Projekte braucht eine ausdrückliche Regel und Migration; das Umhängen eines Projekts ist kein stiller Zugriffserweiterungsmechanismus.

## 6. Offline und gemeinsames Arbeiten konkret lösen

### 6.1 Rechte, Verfügbarkeit und Reservierung bleiben unabhängig

Pro Dokument werden Zugriff, tatsächliche lokale Verfügbarkeit, führendes Speichersystem, Bearbeitungs-/Reservierungszustand und erlaubte KI-Verarbeitung getrennt geführt. Ein Download erteilt keine zusätzlichen Rechte, eine offene Ansicht reserviert nichts, und Dateisync macht kein Cloud-Modell offline ausführbar.

**Offline-Arbeit ist kein gesonderter Pflichtablauf.** Bereits vollständig lokal vorhandene und zur Offline-Nutzung erlaubte Inhalte bleiben bei Verbindungsverlust verwendbar. Lokale Änderungen werden gesichert und später abgeglichen. Native lokale Projekte benötigen keinen vorbereitenden Download. „Offline behalten“ ist eine zusätzliche Möglichkeit für Cloud-/DMS-Inhalte: vollständig herunterladen und nicht automatisch aus dem Cache entfernen. Noch ausstehende Downloads sind sichtbar. Ein Projekt darf nicht pauschal als vollständig offline verfügbar erscheinen, wenn Inhalte fehlen.

### 6.2 Festgelegte Bedeutung von Sichern, Speichern und Teilen

Der Plan unterscheidet die automatische **Wiederherstellungssicherung des Editors** von einem **Speichern in die Arbeitsdatei**. Die Wiederherstellungssicherung schützt den aktuellen Entwurf außerhalb des normalen Dateisyncs; sie wird nicht allein durch diese Sicherung zur Teamversion. Sie ersetzt keine Sicherung des gesamten Geräts und ist kein allgemeines Versprechen über andere Cloud-/Backup-Funktionen.

| Betriebsart | Was bedeutet Speichern? | Was wird gemeinsam verfügbar? |
|---|---|---|
| Rein lokales Projekt | Manuelles Speichern oder aktiviertes Autosave aktualisiert die lokale Datei. | Ohne Freigabe kein Team-Sync. |
| Natives geteiltes Projekt | Speichern aktualisiert die Projektdatei und stellt ihre Änderung für den Sync bereit. Aktiviertes Autosave hat dieselbe Bedeutung. | Die Änderung wird nach erfolgreicher Versions- und Rechteprüfung automatisch übertragen; kein zusätzlicher Freigabeklick pro normalem Speichern. |
| DMS-/angebundenes Dokument mit Arbeitskopie | Speichern und Autosave sichern zunächst die kontrollierte lokale Arbeitskopie. | „Im DMS speichern“ beziehungsweise „Zurückspeichern“ erzeugt nach Prüfung die neue Fassung im führenden System. |

**Eine gespeicherte Datei im geteilten Projektordner ist keine private Entwurfsablage.** „Lokal gespeichert – Übertragung ausstehend“ beschreibt dort nur einen Übertragungszustand, etwa offline oder bei einem Konflikt. Die Datei ist zur Weitergabe vorgesehen. Ein künftig ausdrücklich privater Entwurfsmodus müsste eine getrennte, vom Team-Sync ausgeschlossene Ablage mit denselben fachlichen Schutzbezügen verwenden; er gehört nicht automatisch zum ersten Ausbau.

Diese Entscheidung erhält die normale Arbeit mit Projektdateien. Änderungen durch externe Programme im geteilten Ordner sind ebenfalls zur Synchronisierung vorgesehen. Der vorhandene Datei-Autosave ist dabei von einer zukünftigen durchgängigen Wiederherstellungssicherung zu unterscheiden; deren Schutz muss für jeden unterstützten Editor nachgewiesen werden. Autosave wird durch diesen Plan nicht still für alle Nutzer eingeschaltet. Die Oberfläche zeigt bei seiner Aktivierung, ob es die geteilte Projektdatei oder nur eine DMS-Arbeitskopie aktualisiert.

Gemeinsames Speichern ist keine fachliche Freigabe eines Schriftsatzes und kein Versenden an Mandanten. Solche Freigaben bleiben eigene Vorgänge. Ebenso sind Hintergrundübertragung und Übernahme einer neuen Fassung in einen geöffneten Editor verschiedene Aktionen.

### 6.3 Offene Ansichten, Aktivität und sämtliche Schreiber

**Ein offener Tab reserviert nichts.** Im Zielverhalten erwerben wiederhergestellte oder nur lesende Ansichten keine Schreibzuständigkeit. Die heutige Fensterkoordination fordert sie bereits beim Start an; das ist ein konkreter Änderungsbedarf, kein Widerspruch zur Zielvorgabe.

**Eigene Fenster:** Vor der ersten tatsächlichen Änderung wird eine lokale Schreibzuständigkeit erworben. Eingaben dürfen währenddessen weder verloren gehen noch bereits unkoordiniert in die Datei geschrieben werden. Ein inaktiver, sauberer Editor hält diese Zuständigkeit nicht unnötig fest: Sind keine Änderungen oder laufenden Schreibvorgänge mehr offen, erfolgt die Übernahme ohne Dialog. Existiert dagegen ein anderer Editor mit eigenen Änderungen, wird dessen gesicherter Entwurf ausdrücklich übernommen oder die bisherige Bearbeitung bleibt zuständig. Ein Fokuswechsel veröffentlicht oder verwirft nichts. Eine inzwischen geänderte Basis wird angezeigt, bevor der alte Stand wieder in die Datei geschrieben werden kann. Gemeinsame Live-Editor-Sitzungen in mehreren Fenstern sind ein möglicher späterer Ausbau, keine Voraussetzung.

**Andere Personen und eigene weitere Geräte:** Standard ist ein Hinweis auf bekannte aktuelle Bearbeitung, mit bewusstem Fortsetzen als eigene Fassung. Dieselbe Person auf einem zweiten Gerät wird wie ein weiterer Schreiber behandelt; Gerät und Sitzung sind unterscheidbar. Harte Reservierungen bleiben ein gezieltes Opt-in beziehungsweise eine Vorgabe des DMS. Unveröffentlichte Entwürfe allein verursachen keine unbefristete Team-Sperre. Welche Kanzleirichtlinien künftig eine Reservierung verlangen dürfen, bleibt eine gesonderte Produktentscheidung.

Dafür wird ein neuer, zugriffsgeschützter Aktivitätskanal benötigt. Er enthält Dokumentbezug, handelnde Person, Gerät/Sitzung, gegebenenfalls Agent im Auftrag dieser Person, bekannte Basisversion und Ablaufzeit. Ein Hinweis auf ungespeicherte Änderungen wird nur gemeldet, wenn der betreffende Editor diesen Zustand kennt. Reine Dateiüberwachung weiß nicht, ob in einer externen Word-Instanz noch ungespeicherte Eingaben liegen.

Als erste technische Vorgabe kann eine aktive Sitzung etwa alle 20 Sekunden melden und nach spätestens 60 Sekunden ohne Aktualisierung als nicht mehr aktuell gelten; dies ist zu erproben, keine Garantie über tatsächliche Anwesenheit. Ein unberührter Hintergrund-Tab erneuert keinen aktiven Bearbeitungsstatus endlos. Ablauf der Anzeige löscht keinen Entwurf. Die Aussage lautet dann „keine aktuelle Aktivität bekannt“, niemals „niemand bearbeitet“. Eine offline begonnene Änderung ist vor Wiederverbindung für andere unbekannt. Der Aktivitätskanal überträgt keine Dokumentinhalte und erteilt keine Zugriffsrechte.

**Agenten und externe Programme:** Ein Agent ist ein Schreiber mit Identität und Auftraggeber. Er darf nicht über einen anderen Schreibweg die Basisprüfung oder eine Reservierung umgehen. Bearbeitet ein Mensch gerade, erzeugt der Agent standardmäßig eine Vorschlagsfassung mit Quellenbezug. Externe Programme, die über Legal Work geöffnet werden, benötigen eine überwachte Arbeitskopie mit festgehaltener Basis. Direkte Schreibzugriffe beliebiger Programme in einen frei zugänglichen Ordner lassen sich nicht durch eine reine UI-Sperre kontrollieren; die Grenzen stehen in Abschnitt 6.4.

**Neue Version bei geöffneter Ansicht:** Auch ein sauberer Editor bleibt auf seiner angezeigten Fassung, bis die Person „vergleichen“ oder „übernehmen“ wählt. Bei eigenen Änderungen bleiben Entwurf und Basis erhalten. Eingehende Dateien dürfen im Hintergrund bereitgestellt und ungeöffnete Dateien normal aktualisiert werden; ein Editor muss dafür eine eigene, beständige Basis halten und darf nicht aus der gerade vom Sync ersetzten Datei unbemerkt nachladen. Eine blockierte Veröffentlichung beendet nicht die lokale Entwurfssicherung. Echtes Co-authoring wäre ein ausdrücklich erkennbarer Modus mit anderer Aktualisierungslogik.

### 6.4 Dokumentidentität und Veröffentlichung als eigener Architekturbaustein

**Stabile Dokument-IDs bleiben auch für native Legal-Work-Dateien das Ziel.** Ein Pfad ist ein veränderlicher Ablageort; ein Inhalts-Hash prüft Inhalt und Übertragung. Beides ersetzt keine dauerhafte Dokumentidentität. Eine Wiederherstellung eines früheren Inhalts erzeugt eine neue Version mit neuem Ereignis und Herkunft, auch wenn der Inhalts-Hash schon einmal vorkam.

Der Versionsvertrag umfasst mindestens:

- Dokument-ID, unveränderliche Versionsreferenz, vorherige beziehungsweise bearbeitete Basis, Inhalt/Prüfsumme, Person oder Agent, Zeitpunkt und Quellenbezug.
- Eine Prüfung von aktuellem Zugriff, relevanter Reservierung und erwarteter Basis beim verbindlichen Commit. Prüfung und Annahme dürfen nicht durch einen dazwischenliegenden Schreibvorgang oder Rechteentzug auseinanderfallen.
- Eine dauerhafte Operations-ID pro Speichervorgang. Nach Abbruch oder verlorener Bestätigung kann derselbe Vorgang wiederholt oder abgefragt werden, ohne eine zweite Veröffentlichung zu erzeugen. Ein unvollständiger Upload ist noch keine sichtbare Version.
- Einen erhaltenen lokalen Entwurf bei Abweisung, Vergleich mit der gemeinsamen Basis und eine ausdrücklich erzeugte Konfliktauflösung. Auch „meine Fassung verwenden“ prüft den dann aktuellen Stand erneut; es ist kein allgemeiner Force-Overwrite.
- Umbenennen/Verschieben als Änderung der Ablagezuordnung und Löschen als nachvollziehbaren Zustand. Gleichzeitiges Bearbeiten und Löschen wird sichtbar geklärt; Versionen und Entwürfe verschwinden dabei nicht still.

Das erfordert Änderungen an Plattformvertrag, lokalem Dokumentregister, Sync, Editorbasis, Wiederherstellung und Fensterkoordination. Es ist **kein kleines Ergänzen von Metadaten**. Übertragung, Warteschlangen und Prüfsummen des bisherigen Syncs können weiterverwendet werden; seine pfadbasierten Abgleichs- und Konfliktregeln müssen gezielt weiterentwickelt werden.

Für Umbenennungen außerhalb der App braucht es einen Pfad-ID-Abgleich. Identischer Inhalt allein beweist keine Identität: Eine Kopie kann denselben Hash haben, und ein umbenanntes Dokument kann gleichzeitig verändert werden. Nur eindeutig belegte Vorgänge werden automatisch zugeordnet; unklare Fälle werden zur Entscheidung vorgelegt. Eine vorgeschaltete Historie je Pfad kann ein begrenzter Schutzgewinn sein, darf aber nicht als Erfüllung der Dokumentidentitätsziele verkauft werden.

**Grenze frei zugänglicher Ordner:** Zwei beliebige externe Programme können zwischen Beobachtungen dieselbe Datei überschreiben. Ein Watcher und eine Serverprüfung können eine zuvor unbemerkt verlorene lokale Fassung nicht wiederherstellen. Die starke Zusage „keine verlorenen konkurrierenden Änderungen“ gilt daher für kontrollierte Legal-Work-Schreibwege und integrierte externe Arbeitskopien. Direkter Zugriff im Dateisystem bleibt möglich, erhält aber nur den nachgewiesenen Beobachtungs-/Sicherungsschutz. Eine strengere Garantie benötigt einen kontrollierten Speicherzugang; sie wird nicht ohne Nachweis behauptet. Dasselbe gilt für Speicheradapter ohne atomare Versionsvorbedingung gegenüber fremden Schreibern.

### 6.5 Verhalten in den entscheidenden Konfliktfällen

| Situation | Zielverhalten |
|---|---|
| Viele offene Tabs, keine Bearbeitung | Keine Reservierung; Lesen und spätere Bearbeitung in anderen Fenstern bleiben möglich. |
| Zwei eigene Fenster beginnen zu ändern | Lokale Koordination vor dem Schreiben; sichere Übergabe oder getrennte Vorschlagsfassung, kein verworfener Entwurf. |
| Zwei Personen oder zwei eigene Geräte ändern dieselbe Basis | Aktivität ist nur Hinweis. Die verbindliche Versionsprüfung nimmt einen Stand an; der andere bleibt als eigene Fassung zur Entscheidung erhalten. |
| Eine Person arbeitet offline, eine andere veröffentlicht online | Beide Fassungen und die gemeinsame Basis bleiben erhalten. Kein automatisches Überschreiben bei Wiederverbindung. |
| Eine neue gemeinsame Version trifft bei geöffneter Ansicht ein | Anzeige „Neue Version verfügbar“; weder saubere Ansicht noch eigener Entwurf werden still ersetzt. |
| Agent arbeitet an einem vom Menschen bearbeiteten Dokument | Vorschlagsfassung statt Überschreiben; Auftraggeber und Basis sind erkennbar. |
| Speichern wurde angenommen, die Bestätigung geht verloren | Wiederholung derselben Operations-ID bestätigt denselben Commit statt einer zweiten Veröffentlichung. |
| Reservierung läuft ab oder wird aufgehoben | Entwurf bleibt erhalten; spätere Veröffentlichung prüft die aktuelle Berechtigung, Reservierungsgeneration und Basis erneut. |
| Datei wird umbenannt oder gelöscht, während ein anderes Gerät sie ändert | Stabile Identität beziehungsweise ausdrückliche Zuordnungsentscheidung; kein stilles Duplikat und kein Verlust der Bearbeitung. |

Eine verbindliche Reservierung kann nur online bei der maßgeblichen Instanz erworben werden. Offline entsteht daraus keine neue globale Exklusivität. Reservierungsdauer, Verlängerung und protokollierte Aufhebung sind getrennt von der kurzlebigen Aktivitätsanzeige. Verlangt das DMS einen Checkout, gelten dessen Regeln; Legal Work simuliert keine unabhängige zweite Sperrverwaltung.

### 6.6 Führendes System und Anbieterfähigkeiten

**Native Legal-Work-Dokumente:** Die Plattform führt die gemeinsame, versionierte Fassung; lokale Dateien bleiben die zugängliche Arbeitsdarstellung. Der Ausbau aus Abschnitt 6.4 ist dafür erst umzusetzen. Ein bereits existierender Dateisync ist kein Nachweis eines vollständigen Versionsarchivs.

**DMS-Dokumente:** Das DMS führt Rechte, Identität und Versionen. Legal Work hält stabile Referenzen, zulässige Metadaten und Arbeitskopien. Bibliothek/Repository, Dokument-ID, Version und Aktenbezug bleiben erhalten. DMS-Arbeitskopien werden vom zweiten Team-Dateisync ausgeschlossen. Zurückspeichern, neue eigenständige Datei und private Wiederherstellungssicherung sind unterscheidbare Vorgänge.

**Einfache Dateispeicher:** Ein generischer Adapter ohne DMS-Fähigkeiten bleibt nutzbar. Er wird nicht als vollwertige Akten-, Reservierungs- oder Versionsintegration dargestellt. Pro Anbieter werden Identität, Rechteprüfung, Versionsvorbedingung, Historie, Reservierung, Änderungserkennung und Offline-Arbeitskopie als unterstützte oder fehlende Fähigkeiten geführt.

Connectoren handeln mit der Identität und den Rechten der jeweiligen Person. Allgemein berechtigte Sammelkonten dürfen persönliche Informationsbarrieren nicht aufheben. Die bestehenden in der Anwendung engeren Regeln für Teamkonten ersetzen keine benutzerbezogene DMS-Authentifizierung. [Dokumentierte Team-Speichergrenzen](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/docs/team-storage.md).

### 6.7 Rechteentzug, geschützte Übergabe und spätere Cache-Option

Beim Rechteentzug stoppen verbundene Geräte weitere zulässigkeitsabhängige Zugriffe. Ein vollständig getrenntes Gerät kann nicht sofort erreicht werden. Bei normalen Klartextdateien lässt sich außerdem keine verbindliche Offline-Ablauffrist allein durch die App durchsetzen. „Zugriff beendet“, „von der App-Kopie entfernt“ und „sämtliche Kopien gelöscht“ sind verschiedene Aussagen; die letzte ist für Exporte und eigene Ordner nicht belegbar.

Für kanzleiverwaltete Daten ist ungesendete Arbeit geschützt an berechtigte Verantwortliche zu übergeben. Dazu braucht es einen **separaten, eng begrenzten Annahmekanal**, der keine erneuten Lese-, Listen- oder Projektbearbeitungsrechte erteilt. Eine Übergabeberechtigung wird an Organisation, Identität, zulässige Dokumentbezüge, Zweck und Laufzeit gebunden. Sie nimmt eine unbestätigte Fassung in Quarantäne auf, veröffentlicht sie nicht als gültige Dokumentversion. Die Übernahme durch eine berechtigte Person wird protokolliert. Fehlt nach Kontosperre ein authentifizierbarer Übergabekanal, bleibt eine gesicherte lokale Wiederherstellung mit administrativ organisiertem Übergabeweg erforderlich; vollständige Automatik darf dann nicht versprochen werden.

Bestehende eigene Projektordner werden durch eine neue Mandantenzuordnung weder enteignet noch gelöscht. Für deren Umwandlung in einen streng verwalteten Bereich braucht es eine ausdrückliche Migration mit klaren Kopier-/Exportgrenzen. Bis dahin wird die begrenzte Rückrufwirkung offen angezeigt. Organisationsbesitz und die nötigen Administrationsrechte sind Voraussetzung für die neue Übergabelogik.

Ein gerätegebundener, verschlüsselter Cache mit Offline-Berechtigungsdauer bleibt eine **spätere gesonderte Option**, zunächst besonders für DMS-Arbeitskopien. Er ist nicht Voraussetzung für die Mandantenübersicht. Vor Einführung sind externe Office-Bearbeitung, Agentenzugriff, Wiederherstellung und Schlüsselverwaltung nachzuweisen. Ein solcher Cache ist grundsätzlich mit automatischer Offline-Arbeit vereinbar; er erfordert aber einen eigenen Integrationsentwurf. Vollverschlüsselung des Geräts kann Schutz bei Geräteverlust beitragen, ersetzt weder Dateirechte noch einen Rückruf bereits exportierter Inhalte.

## 7. Zeiterfassung und spätere Kanzleifunktionen vorbereiten

Abrechenbare Zeiterfassung sollte an das **Mandat**, die leistende Person und gegebenenfalls Projekt und konkreten Arbeitsvorgang gebunden sein. Interne Zeiten können einem internen Projekt ohne Mandant zugeordnet werden und benötigen keinen künstlichen Kunden. Mandant, Rechnungsempfänger und Billing Partner sind getrennte Referenzen. Eine Leistung benötigt mindestens Datum, Dauer/Menge, Tätigkeit, Beschreibung, Verrechenbarkeit, Status und Herkunft. Sätze und Tarifregeln brauchen eigene Versionen beziehungsweise den bei Freigabe festgehaltenen Stand.

Sinnvoller Ablauf:

**Entwurf → eingereicht → geprüft/freigegeben → abgerechnet → gegebenenfalls nachvollziehbar korrigiert.**

Ein Associate kann eigene Zeiten erfassen, ohne fremde Honorarsätze zu sehen. Billing kann Leistungen aufbereiten; der zuständige Partner genehmigt den vorgesehenen Umfang. Bereits abgerechnete Leistungen werden nicht unbemerkt überschrieben. Für Finanzdaten ist „die letzte Änderung gewinnt“ kein ausreichendes Sync-Verfahren.

Offline-Erfassung erzeugt lokale, eindeutig identifizierte Ereignisse. Beim Wiederverbinden werden sie genau einmal übernommen. Änderungen an bereits eingereichten oder freigegebenen Positionen lösen eine kontrollierte Revision aus. Mehrere Geräte dürfen nicht dieselben Minuten doppelt abrechnen. KI kann aus Arbeitsschritten Leistungsvorschläge erzeugen, aber Laufzeit eines Agenten ist nicht automatisch abrechenbare Anwaltszeit.

Weitere anschlussfähige Bereiche sind Mandatsannahme und Kollisionsprüfung, Kontakte/Beteiligte, Budgets, Fristen mit Vier-Augen-Freigabe, Korrespondenzzuordnung, Mandantenportale, Signaturen, Archivierung und Aufbewahrung. Bestehende Buchhaltung, ERV und Honorarlogik würde ich zunächst anbinden. Österreichische Abrechnung und Gerichtskommunikation sind eigenständige Fachprodukte; sie sollten nicht beiläufig als Teil des ersten Organisationsumbaus entstehen.

## 8. Umsetzung in abgegrenzten Etappen

### Schritt 1: Mandantenübersicht mit belastbarer fachlicher Identität

Die vorhandene Organisation wiederverwenden. Mandanten, Mandate, deren Mandantenbeziehungen und die optionale Mandatsreferenz am Projekt von Anfang an mit stabilen IDs führen. Die Oberfläche bietet „vorhandener Mandant“, „neuer Mandant“ und „kein Mandant“; ein vorhandenes Mandat kann ausdrücklich gewählt werden. Im einfachen Fall entsteht das Mandat im Hintergrund. Keine zusätzliche Pflichtmaske, kein Pseudomandant, keine automatische Zusammenlegung gleicher Namen.

Dazu gehören die Sidebar-Ansichten, die auf sichtbare Projekte begrenzte Stammdatensicht, lokale Replik und Ausgangswarteschlange aus Abschnitt 5.4. Die Zuordnung bleibt unabhängig vom optionalen Projektdetails-Sync. Eine erste Zuordnung bestehender Projekte kann manuell erfolgen; ein intelligenter Migrationsassistent ist kein notwendiger Bestandteil dieses Schritts. Projektreferenzen, Dateipfade und bestehende Freigaben ändern sich nicht still.

**Abgrenzung:** Neue fachliche Struktur, noch keine neue Vererbung von Mandatsrechten, kein Dokumentversionssystem, keine Anwesenheitsanzeige und keine Finanzverwaltung. Die Projektberechtigungen schützen auch die erste Navigation. Eine Organisationstabelle darf nicht versehentlich zum frei zugänglichen Verzeichnis aller Mandate werden.

Vor der Umsetzung ist der Vertrag im tatsächlichen `model-api`-Backend zu prüfen. Die Zahl benötigter Tabellen, Endpunkte und Migrationen lässt sich aus dem nicht zugänglichen Backend nicht seriös festlegen; Fables Schätzung „zwei Tabellen und ein Projektattribut“ wird nicht übernommen.

### Schritt 2: Kanzleibesitz, Teams und Rechte verbindlich machen

Kanzleimandate organisatorisch besitzen lassen, mit übertragbarer Verantwortung. Projekt-Ersteller, verantwortlicher Partner und Berechtigungsverwaltung bleiben getrennt. Das Ausscheiden einer Person darf keine Akten verwaisen lassen. Die Migration bestehender persönlicher Projektverantwortung erfolgt ausdrücklich.

Mandatsrollen und projektspezifische Einschränkungen erhalten einen dokumentierten Vererbungsvertrag. Bestehende Freigaben werden nicht durch die Zuordnung zu einem gemeinsamen Mandat erweitert. Lesen, Bearbeiten, Teamverwaltung, Teilen, Offline-Kopien, Finanzdaten und Aufheben einer Reservierung werden getrennt geprüft. Die Schutzentscheidung gilt für API, Sync, Suche, Export und Agenten. Hier werden auch der geschützte Übergabekanal und der Umgang mit Rechteentzug aus Abschnitt 6.7 spezifiziert; eine strengere Rückrufgarantie wird erst nach entsprechender Abnahme angeboten.

### Schritt 3A: Editorverhalten und sichere lokale Wiederherstellung

Kann parallel zur fachlichen Navigation beginnen. Wiederherstellungssicherung und Veröffentlichung klar trennen. Aktive Ansichten auf ihrer Basis halten, eingehende Versionen anzeigen, Entwürfe bei Konflikt und Absturz erhalten. Lokale Schreibzuständigkeit erst bei tatsächlicher Bearbeitung erwerben, sichere Übergabe zwischen eigenen Fenstern prüfen. Agenten sollen bei aktiver menschlicher Bearbeitung Vorschlagsfassungen erzeugen.

Die verbleibenden Grenzen des alten Dateisyncs und frei zugänglicher Ordner bleiben sichtbar. Diese Etappe behauptet weder ein neues unveränderliches Serverarchiv noch vollständige Identitätserhaltung bei externen Umbenennungen. Temporäre bestehende Pfadschlüssel werden bei Einführung der Dokument-IDs nachvollziehbar migriert; aktive Entwürfe dürfen dabei nicht verloren gehen.

### Schritt 3B: Versions- und Identitätsvertrag für native Dokumente

Eigener technischer Entwurf und eigener Implementierungsumfang: Dokumentregister, IDs, unveränderliche Versionen, idempotente Veröffentlichungen, Rechte-/Basisprüfung, Pfadzuordnung, Konflikt- und Löschzustände. Erst nach Prüfung der tatsächlichen Plattformfähigkeiten den Umbau von Client und Backend zuschneiden. Die Vorgaben aus Abschnitt 6.4 gelten als Abnahmekriterien.

Die Migration braucht einen Pilot mit bestehenden Projekten und einen Wiederherstellungsweg. Alte und neue Clients dürfen dieselbe Datei nicht über widersprüchliche Commit-Verträge bearbeiten; je Projekt wird vor Aktivierung geklärt, welche Clientversionen weiter schreiben dürfen. Ein versionsunfähiger Client wird gegebenenfalls auf Lesen beschränkt oder benötigt ein Update. Keine stille Umschaltung laufender Projekte.

Der neue Aktivitätskanal wird auf dieselbe Dokumentidentität gesetzt. Reservierungen sind ein eigener optionaler Vertrag, keine Verlängerung des Aktivitätsstatus. Der reine Transport kann weitgehend wiederverwendet werden; Konsistenzregeln und Historie werden ausdrücklich neu geprüft.

### Schritt 3C: Einen vollständigen DMS-Pfad liefern

Ein priorisiertes DMS mit persönlicher Identität, Quellenrechten, stabilen Dokument-/Aktenreferenzen, Versionen, Arbeitskopien und geprüftem Zurückspeichern vollständig anbinden. Fehlende Providerfähigkeiten offen ausweisen. Native Teamreplikation und DMS-Sync dürfen dieselbe Arbeitskopie nicht konkurrierend veröffentlichen. Bestehende generische Speicheradapter bleiben als einfachere Integrationen verfügbar.

Ein streng verwalteter Cache ist ein separater späterer Ausbau mit eigener Abnahme für Schlüsselverwaltung, externe Bearbeitung, Offline-Berechtigungsdauer und Wiederherstellung. Er blockiert weder Schritt 1 noch den normalen nativen Dateisync.

### Schritt 4: Leistungserfassung und Finanzintegration

Baut auf stabilen Mandatsidentitäten und Finanzrechten auf, benötigt aber nicht den Abschluss sämtlicher Dokument-/DMS-Arbeiten. Zunächst lokale/offline Erfassung mit kontrollierter Übertragung in das führende Kanzleisystem, danach Freigaben, Budgets und Auswertungen. Leistungen halten ihren ursprünglichen Mandatsbezug auch beim späteren Umhängen eines Projekts; eine Korrektur ist ein eigener protokollierter Vorgang. Welche Abrechnungsfunktionen Legal Work selbst führt, richtet sich nach konkreten Kanzleibedarfen.

### Schritt 5: Gemeinsames Bearbeiten und Cloud-Ausführung erweitern

Co-authoring, externe Portale und autonome Cloud-Aufträge verwenden dieselben Identitäten, Schreibverträge und Rechte. Vor einem Cloud-Auftrag und vor der Annahme seiner Ergebnisse werden Berechtigung und Datenfreigabe erneut geprüft. Ein persönlicher Cloud-Assistent ist weder ein zweites Berechtigungssystem noch ein Weg um Versionsprüfungen.

**Abhängigkeiten:** Schritt 1 → Schritt 2 → Finanzfunktionen; Editorverbesserungen können parallel starten. Das neue native Versionssystem benötigt den geprüften Plattformvertrag, die vollständige DMS-Anbindung ihren eigenen Anbieter- und Rechtevertrag. Die Mandantenübersicht wartet nicht auf ein neu gebautes DMS.

## 9. Akzeptanzszenarien je Etappe

Diese Szenarien sind Vorgaben für die Umsetzung, keine bereits ausgeführten Tests. Freigaben erfolgen je Etappe; der erste Sidebar-Ausbau muss nicht sämtliche späteren DMS- und Finanzkriterien erfüllen.

| Etappe | Szenario und erwartetes Ergebnis |
|---|---|
| 1 | Zwei Projekte desselben Mandanten mit verschiedenen Teams: jedes Mitglied sieht nur zulässige Projekte, Namen und Zähler. Wechsel der Sidebar-Gruppierung verändert keine Rechte. |
| 1 | Internes, mit der Kanzlei geteiltes Projekt: Mandats-/Mandantenzuordnung bleibt leer; Teilen erzeugt keinen künstlichen Mandanten. |
| 1 | Zwei Projekte desselben Mandats und zwei Mandanten dieses Mandats: eindeutige Referenzen, keine kopierten Projekte, keine widersprüchlichen unabhängigen Mandantenfelder. |
| 1 | Projektdetails-Sync ausgeschaltet: berechtigte Empfänger erhalten die fachliche Zuordnung trotzdem. |
| 1 | Offline neuen Mandanten und neues Projekt anlegen; Übertragung mehrfach unterbrechen: ein Mandant, ein Mandat und eine Zuordnung nach erfolgreicher Annahme. Namensgleiche Datensätze werden nur bei berechtigter Sicht zur Zusammenführung vorgeschlagen. Abgelehnte Vorgänge bleiben als lokale, nicht veröffentlichte Entwürfe erkennbar. |
| 2 / 4 | Projekt in ein anderes Mandat umhängen: keine stille Teamvereinigung; bereits erfasste Leistungen behalten ihren alten Mandatsbezug. |
| 3A | 17 wiederhergestellte Arbeitsbereiche: unberührte Dokumente bleiben unreserviert. Ein sauberes inaktives Fenster erzwingt keinen Übernahmedialog. Bearbeitung im zweiten eigenen Fenster und eine Übergabe verlieren auch bei Absturz keinen bereits gesicherten Entwurf. |
| 3A | Neue Fassung trifft bei sauberer oder geänderter offener Ansicht ein: Hinweis statt stiller Ersetzung; Vergleichen und Übernehmen erhalten die eigene Arbeit. |
| 3B | Zwei Teammitglieder oder dieselbe Person auf zwei Geräten bearbeiten dieselbe Basis, eines offline: beide Fassungen bleiben erhalten; die zweite Veröffentlichung löst eine sichtbare Entscheidung aus. Eine abgelaufene Aktivitätsanzeige gilt nicht als Nachweis exklusiven Zugriffs. |
| 3A / 3B | Mensch, Agent und integrierte externe Arbeitskopie arbeiten am gleichen Dokument: Autor/Basis sind erkennbar, kein Umgehen des Commit-Vertrags, Entwürfe bleiben erhalten. Direkte unkontrollierte Dateisystemschreiber werden getrennt auf die tatsächlich zugesagte Schutzwirkung geprüft. |
| 3B | Server nimmt eine Veröffentlichung an, Bestätigung geht verloren: Wiederholung mit gleicher Operations-ID erzeugt genau eine angenommene Version. Eine Wiederherstellung früheren Inhalts erzeugt dagegen eine neue Version. |
| 3B | Kontrolliertes Umbenennen eines Dokuments oder Ordners bei paralleler Bearbeitung: Identität bleibt bestehen, keine irreführende Massenlöschung. Bei unklarer externer Zuordnung und identischen Dateikopien wird keine Identität allein aus dem Hash geraten. Löschen gegen Bearbeiten endet in einer sichtbaren Entscheidung mit erhaltener Arbeit. |
| 2 / 3B | Reservierung aufheben, Rechte entziehen oder Benutzer sperren, während ein Commit läuft: eindeutig festgelegte Reihenfolge, keine Veröffentlichung nach wirksamem Entzug. Alte Reservierungsinhaber überschreiben keinen neueren Stand. |
| 2 | Billing sieht erforderliche Kennungen und Leistungen, aber keine gesperrten Inhalte. Suche, Vorschauen, Agenten, Wiki-Ableitungen, Exporte und Benachrichtigungen umgehen diese Grenze nicht. |
| 2 / 3C | Rechteentzug mit ungesendeter Arbeit: begrenzter Übergabekanal nimmt nur eine Quarantänefassung an, gibt keine Projekt-Leserechte zurück. Eigene Ordner/Exporte werden nicht als erfolgreich zurückgerufen ausgegeben. Offline-Zusagen entsprechen der tatsächlich verwendeten Speicherart. |
| 3C | DMS-Dokument umbenennen, bearbeiten und zurückspeichern: richtige Identität, Version und persönliche Rechte; kein zweiter Upload über nativen Team-Sync. |
| 2 | Verantwortlicher Partner verlässt die Kanzlei: Akten, Projekte und Zuständigkeiten bleiben organisatorisch übernehmbar. |
| 4 | Offline erfasste Leistungen wiederholt synchronisieren: keine doppelte Verbuchung; freigegebene und abgerechnete Positionen ändern sich nur nachvollziehbar. |

## 10. Festgelegte Richtung und verbleibende Entscheidungen

Für den überarbeiteten Plan sind festgelegt: frühe Mandatsidentität mit einfacher Oberfläche; mehrere Projekte pro Mandant und pro Mandat; interne Projekte ohne künstlichen Mandanten; organisationsbezogener, zugriffsgefilterter Stamm; automatischer Sync gespeicherter nativer Projektdateien; bewusstes Zurückspeichern von DMS-Arbeitskopien; keine Sperre allein durch Öffnen; beständige geöffnete Ansicht mit Hinweis auf neue Versionen; stabile Dokumentidentitäten auch für native Dateien als eigener Ausbau.

Standard für Teamarbeit ist eine zeitlich begrenzte Aktivitätsanzeige mit bewusster paralleler Bearbeitung, ergänzt durch optionale beziehungsweise DMS-vorgegebene Reservierungen. Daraus folgt kein Verbot, später für bestimmte Dokumente eine verbindliche Reservierung zu verlangen. Diese Richtlinie wird nicht durch einen dauerhaft offenen Tab ausgelöst.

Vor den jeweils betroffenen Etappen sind noch festzulegen:

1. **Reservierungsrichtlinien:** Für welche Dokumentarten oder Kanzleien soll eine Reservierung verpflichtend sein, mit welcher Dauer und welchen Aufhebungsrechten? Der Standardablauf bleibt ohne Öffnungssperre.
2. **Erstes DMS und externe Bearbeitung:** Welche konkrete Integration und welche externen Editoren werden mit belastbarem Schreibvertrag zuerst unterstützt? Einfacher Dateizugriff bleibt von dieser Zusage getrennt.
3. **Versionen und Aufbewahrung:** Welche gespeicherten Fassungen erscheinen als benannte Arbeitsstände, wie werden automatische Stände dargestellt, und welche Wiederherstellungs-/Aufbewahrungsfristen gelten? Die interne Commit-Identität und Konfliktsicherung hängen nicht von einer manuellen Versionsbenennung ab.
4. **Sichtbare Mandatsübersicht:** Wann wird die zusätzliche Ebene eingeblendet und wie werden mehrere Mandanten dargestellt? Das zugrunde liegende Beziehungsmodell steht bereits fest.
5. **Streng verwaltete Offline-Daten:** Für welche Mandate ist ein kontrollierter Cache erforderlich, und welche externen Zugriffe sind dann zulässig? Bis zu seiner Abnahme gibt es keine pauschale Rückruf- oder Offline-Ablaufgarantie für normale Projektdateien.

Diese Entscheidungen blockieren nicht die erste Mandantenübersicht. Der konkrete Plattformvertrag und die Migration sind dagegen Voraussetzung für die jeweilige Implementierung; sie müssen anhand des tatsächlichen Backends ausgearbeitet werden.

## 11. Übernahme des Fable-Reviews

Der [unveränderte Fable-Review](Fable-Adversarial-Review.md) bezieht sich auf die [eingefrorene frühere Planfassung](fable-review-plan-snapshot.md). Seine Zeilenangaben gehören zu dieser Fassung. Die folgenden Bewertungen betreffen die Empfehlungen; ein fehlendes Merkmal im heutigen Code ist für sich genommen kein Widerspruch in einem Zielkonzept.

| Finding | Entscheidung im überarbeiteten Plan |
|---|---|
| 1 – Mandant-Link und späteres Mandat | Übernommen: Mandatsidentität ab Schritt 1, optionaler Projektbezug und führende Mandantenbeziehung am Mandat. Keine doppelte Anlage in der Oberfläche. Mehrmandantenfähigkeit, Umhängen und historische Leistungen ausdrücklich berücksichtigt. |
| 2 – Lokal gesichert, noch nicht geteilt | Übernommen und präzisiert: Wiederherstellungssicherung, Speichern der nativen Projektdatei und DMS-Zurückspeichern unterscheiden. Speichern beziehungsweise aktiviertes Datei-Autosave im geteilten Ordner stellt zur Übertragung bereit. Kein heimliches Einschalten von Autosave und kein Privatheitsversprechen für geteilte Ordner. |
| 3 – Aufwand für IDs und Versionen | Aufwandskritik übernommen: eigener Entwurf, Plattformänderung und Migration. Dauerhaft pfadbasierte native Dokumente nicht übernommen; sie lösen die Identitätsziele nicht. Hashvergleich allein genügt auch nicht zur sicheren Umbenennungserkennung. |
| 4 – Mandantenstamm und Offline-Anlage | Übernommen: Organisationsentitäten mit lokalem zulässigem Ausschnitt, Ausgangswarteschlange, stabilen IDs und unabhängiger Projektzuordnung. Keine frei sichtbare Komplettliste aller Kanzleimandanten. |
| 5 – Aktivität und eigene weitere Geräte | Übernommen: neuer zugriffsgeschützter Aktivitätskanal, Gerät/Sitzung, Ablauf und ausdrückliche Unsicherheit. Externen Editoren wird kein unbekannter Dirty-Zustand unterstellt. Reservierungsrechte sind getrennt. |
| 6 – Agent als Schreiber | Übernommen: gleiche Basis-/Rechteprüfung, erkennbare Herkunft und Vorschlagsfassungen bei aktiver menschlicher Bearbeitung; keine Umgehung durch Werkzeuge. |
| 7 – Übergabe nach Rechteentzug | Übernommen und begrenzt: zweckgebundener Annahmekanal ohne wiedererteilte Leserechte, Quarantäne statt Veröffentlichung, administrativer Ersatzweg bei fehlender Authentifizierung. Grenzen eigener Ordner/Exporte bleiben explizit. |
| 8 – Verwalteter Cache | Aufwand und Etappentrennung übernommen. Als spätere Option statt Voraussetzung für die Mandantenübersicht. Die behauptete grundsätzliche Unvereinbarkeit mit automatischer Offline-Arbeit wird nicht übernommen. |
| 9 – Sperre beim Öffnen | Konkreten Änderungsbedarf übernommen: Schreibzuständigkeit erst bei Bearbeitung. Die Abweichung des aktuellen Codes ist Implementierungsarbeit, kein Beweis eines widersprüchlichen Zielmodells. |

Die Herstellervergleiche und ursprünglichen PR-Befunde wurden durch diesen Review nicht neu verifiziert. Für die Überarbeitung wurden die im Review benannten lokalen Sync-, Autosave- und Speicherverträge gezielt gegengelesen. Es wurden keine Produktivdaten verändert und keine Implementierungstests ausgeführt.
