# Legal Work: Kanzleistruktur, Berechtigungen und Zusammenarbeit

**Konzeptstand: 23. Oktober 2026.** Die Empfehlungen wurden anhand des Nutzerfeedbacks zu mehreren Projekten pro Mandant, internen Projekten, Offline-Arbeit und vielen gleichzeitig geöffneten Arbeitsbereichen überarbeitet. Der nachfolgend dokumentierte Code- und Recherche-Snapshot wurde dabei nicht erneut erhoben. Die vorgeschlagene Behandlung konkurrierender Bearbeitung bleibt eine Empfehlung, keine bereits beschlossene oder implementierte Funktion.

**Ergebnis:** Legal Work hat bereits eine Organisationsebene, gemeinsame Projekte und bidirektionalen Dateisync. Was fehlt, ist eine verbindliche fachliche Schicht für **Mandanten, Mandate, Beteiligte, Mandatsteams und Finanzrechte**. Diese sollte auf den vorhandenen Grundlagen aufbauen. Ein zusätzliches Feld „Mandant“ oder ein Ordner oberhalb der Projekte würde das Problem nur optisch lösen.

Die empfohlene Richtung ist: **lokal arbeitsfähig, zentral berechtigt, dokumentbezogen versioniert und mit klarer Zuständigkeit des führenden Systems**. Bei einer iManage-Anbindung bleibt iManage für seine Dokumente und deren Zugriffsgrenzen führend; Legal Work ergänzt Arbeitsabläufe, lokale Entwürfe und KI-Unterstützung.

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

Begrifflich ist der Mandant die Person oder das Unternehmen, das Mandat der konkrete Auftrag/Fall und das Projekt der Arbeitskontext in Legal Work. Häufig deckt ein Projekt genau ein Mandat ab. Dafür sollte die Oberfläche keine doppelte Anlage oder zusätzliche Navigationsebene verlangen. Mehrere Projekte können aber auch zum selben Mandat gehören, etwa „Due Diligence“ und „Verhandlung“. Der erste Ausbau muss mehrere Projekte pro Mandant unterstützen; ob die zusätzliche Mandatsebene schon sichtbar benötigt wird, hängt von solchen tatsächlichen Mehrprojektfällen und den DMS-/Abrechnungsbezügen ab.

Eine Mandatsübersicht kann Beteiligte, Verantwortliche, Projekte, Dokumente, Aufgaben, Fristen und – soweit berechtigt – Budget und Leistungen bündeln. Technische Fenster, Tabs und wiederherstellbare Layouts sind davon unabhängig: Ein zusätzlich geöffneter Workspace erzeugt weder ein neues Mandat noch neue Rechte oder eine Dokumentreservierung.

| Objekt | Aufgabe |
|---|---|
| Kanzlei / Organisation | Sicherheits-, Verwaltungs- und Zuständigkeitsgrenze. |
| Mandant / Person / Unternehmen | Wiederverwendbare Identität, Kontakte, Kennungen, Beziehungen. |
| Mandat / Akte | Fachlicher Auftrag mit Aktenzeichen, Lebenszyklus, Team, Richtlinien und Finanzbezug. |
| Mandatsbeteiligter | Verknüpft eine Person/Organisation in einer Rolle: Mandant, Gegner, Gericht, Versicherer, Kontakt, Rechnungsempfänger. |
| Projekt | Arbeitskontext mit optionaler Mandanten-/Mandatszuordnung; mehrere Projekte je Mandant und gegebenenfalls je Mandat. Auch intern oder noch nicht zugeordnet möglich. |
| Fenster / Tabs / Workspace-Layout | Persönliche Ansicht auf bestehende Projekte und Dokumente; keine fachliche Eigentums- oder Berechtigungsgrenze. |
| Dokument und Dokumentversion | Dauerhafte Identität sowie nachvollziehbare Fassungen; unabhängig vom Dateinamen. |
| Teamzuordnung | Person/Gruppe, Zuständigkeit, Berechtigungsrolle und Gültigkeitszeitraum. |
| Leistung / Abrechnung | Fachliche Finanzobjekte mit eigener Berechtigung und Freigabestatus. |

Der sichtbare Baum ist nicht das vollständige Datenmodell. Ein Mandat kann mehrere Mandanten haben; eine Versicherung kann Rechnungsempfänger sein, ohne Mandant zu sein. Ein Konzern kann mehrere eigenständige Mandanten enthalten. Ein gemeinsamer Konzernknoten darf nicht automatisch alle Mandate für alle Teams öffnen.

**Keine feste Eins-zu-eins-Regel zwischen Mandant und Projekt.** Zunächst erhalten Projekte eine optionale Zuordnung zu einer stabilen Mandantenidentität. Wo eine gemeinsame Akte mehrere Projekte verbindet oder das DMS eine Aktenidentität vorgibt, kommt die Mandatsreferenz hinzu. Dann wird die Mandantenbeziehung aus dem Mandat übernommen beziehungsweise konsistent mit ihm geführt; widersprüchliche unabhängige Zuordnungen sind zu verhindern. Projektname und Mandantenname bleiben getrennt. Eine spätere zusätzliche Gruppierung erfordert weder neue Projekt-IDs noch eine Verschiebung der lokalen Dateien.

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

### 5.3 Ein einheitlicher Entscheidungsweg

Für jeden Zugriff wird dieselbe Entscheidung benötigt – in der Oberfläche, im Sync, bei der Suche, im Export und im Agenten:

**Erlaubte Rollenaktion im passenden Scope + gültige Mitgliedschaft + erlaubte Daten-/Geräterichtlinie + Berechtigung des Quellsystems; keine entgegenstehende Sperre.**

Finanzrechte bekommen einen eigenen Datenumfang. Billing darf beispielsweise Aktennummer, Rechnungsempfänger, notwendige Leistungstexte und Beträge sehen, ohne dadurch Zugang zu Schriftsätzen oder Strategie-Chats zu erhalten. Besonders geschützte Leistungsbeschreibungen können eine nochmals engere Sicht erfordern.

Volltextindizes, Embeddings, Vorschauen, Exporte, Chat-Anhänge und KI-Zusammenfassungen brauchen dieselben Schutzbezüge wie ihre Quellen. Ein Kanzlei-Wiki darf nicht zum Umweg um eine Mandatssperre werden. Für wiederverwendbares Wissen braucht es eine bewusste Freigabe oder Bereinigung; bei mehreren Quellen gilt ohne solche Freigabe die restriktive Kombination ihrer Schutzregeln. Schon Trefferzahlen, Dateinamen und Benachrichtigungen können geschützte Informationen offenlegen.

## 6. Offline und gemeinsames Arbeiten konkret lösen

### 6.1 Rechte, Download und Sperre sind unabhängig

Pro Dokument bzw. Datenbereich sollten getrennt feststehen:

- **Zugriff:** Wer darf welche Aktion ausführen?
- **Verfügbarkeit:** Nur online, bereits vollständig lokal vorhanden oder dauerhaft lokal zu behalten?
- **Speicherzuständigkeit:** Native Legal-Work-Ablage oder angebundenes DMS?
- **Bearbeitungsmodus:** Private Entwurfsfassung, gemeinsam bearbeitbar oder exklusiv reserviert?
- **KI-Nutzung:** Welche Modelle und welche Ausführungsorte dürfen diesen Inhalt verarbeiten?

**Offline-Arbeit ist kein gesonderter Pflichtablauf.** Bereits vollständig lokal vorhandene, zur Offline-Nutzung erlaubte Inhalte bleiben bei Verbindungsverlust verwendbar. Lokale Änderungen werden zuverlässig gesichert und nach Wiederverbindung abgeglichen. Native lokale Projekte benötigen keinen vorbereitenden Download. Eine optionale Funktion „Offline behalten“ sorgt bei Cloud-/DMS-Inhalten dafür, dass ausgewählte Dateien vollständig heruntergeladen und nicht aus dem Cache entfernt werden. Die Anzeige unterscheidet tatsächliche Verfügbarkeit von noch ausstehenden Downloads. Noch nie heruntergeladene Inhalte sind ohne Verbindung nicht verfügbar; Cloud-KI wird durch Dateisync ebenfalls nicht offline ausführbar.

Die frühere Empfehlung eines ausdrücklichen Flugzeugablaufs war zu weitgehend. Sinnvoll ist eine optionale Verfügbarkeitskontrolle, keine zwingende Vorbereitung und keine automatische Reservierung vor jeder Offline-Arbeit.

### 6.2 Geöffnet, bearbeitet, lokal gesichert und gemeinsam verfügbar

**Ein offener Tab reserviert nichts.** Viele offene Arbeitsbereiche müssen möglich sein, ohne Dokumente für andere zu blockieren. Dafür sind getrennte Zustände nötig:

| Zustand | Bedeutung und empfohlenes Verhalten |
|---|---|
| Geöffnet / gelesen | Keine Schreibreservierung. Weitere Personen und eigene Fenster dürfen das Dokument ebenfalls öffnen. |
| Aktuell in Bearbeitung | Zeitlich begrenzte Aktivitätsanzeige, zum Beispiel „Matteo bearbeitet“. Sie ist ein Hinweis, keine Garantie exklusiver Bearbeitung. |
| Änderungen noch nicht lokal gesichert | Schutz vor Datenverlust in diesem Editor; möglichst sofort dauerhaft als Entwurf sichern. |
| Lokal gesichert, noch nicht geteilt | Die eigene Fassung ist wiederherstellbar, aber noch keine neue gemeinsame Dokumentversion. Dieser Zustand kann auch offline bestehen. |
| Neue gemeinsame Version verfügbar | Die neuere Fassung darf im Hintergrund bereitgestellt werden. Eine bereits geöffnete Ansicht oder ein eigener Entwurf wird dadurch nicht still ersetzt. |
| Ausdrücklich reserviert / im DMS ausgecheckt | Bewusste Exklusivität gemäß verbindlicher Reservierung oder Regeln des führenden DMS. Lesen bleibt möglich, soweit dessen Regeln es erlauben. |

„Ungespeichert“ allein ist kein tragfähiger Auslöser einer teamweiten Dauersperre: Autosave kann den lokalen Zustand sofort sichern, obwohl die Arbeit noch läuft. Umgekehrt kann ein unfertiger Entwurf tagelang liegen bleiben. Die Aktivitätsanzeige wird durch aktuelle Lebenszeichen erneuert; ohne diese wird sie als veraltet beziehungsweise offline kenntlich gemacht. Ein inaktiver Tab hält sie nicht unbegrenzt aufrecht. Das Ende der Anzeige verwirft niemals den Entwurf. Änderungen, die erst offline beginnen, sind den anderen naturgemäß noch nicht bekannt.

**Eigene Fenster:** Dieselbe Dokumentidentität erhält eine koordinierte lokale Bearbeitung. Mehrere Ansichten sind zulässig; eine zweite unabhängige schreibende Sitzung wird verhindert oder ausdrücklich übergeben. Die bestehende lokale Dokument-Ownership ist dafür ein Ansatzpunkt. Eine Übergabe übernimmt den gesicherten Entwurf; sie darf ihn weder verwerfen noch allein durch Fokuswechsel eine Veröffentlichung auslösen. Ein bloß wiederhergestellter Tab beansprucht keine Schreibzuständigkeit. Alternativ können mehrere Fenster dieselbe Editor-Sitzung darstellen, sofern deren Zustandsabgleich zuverlässig umgesetzt ist.

**Verschiedene Teammitglieder:** Als Standard empfehle ich einen sichtbaren Bearbeitungshinweis mit ausdrücklicher Möglichkeit, eine eigene parallele Fassung zu bearbeiten. Eine verpflichtende Sperre beim ersten Ändern ist eine noch offene Produktentscheidung, kein bereits akzeptierter Nutzerwunsch. Harte Reservierungen bleiben gezielt verfügbar oder werden verwendet, wenn das DMS sie verlangt. Eine Warnung ersetzt keine technische Konfliktsicherung.

**Änderungen anderer:** Bei einer geöffneten Version erscheint etwa „Neue Version von Anna verfügbar – vergleichen / übernehmen“. Ohne eigenen Entwurf kann die Person bewusst aktualisieren. Mit eigenen Änderungen werden beide Fassungen und ihre gemeinsame Basis erhalten; „übernehmen“ darf den Entwurf nicht still löschen. Noch nicht geöffnete Dokumente können normal auf den aktuellen gemeinsamen Stand synchronisiert werden. Der bloße Hintergrunddownload einer neuen Version ist von deren Übernahme in eine aktive Ansicht zu unterscheiden. Echtes Co-authoring wäre ein eigener, erkennbarer Modus mit bewusst anderem Verhalten.

**Veröffentlichung:** Der Server beziehungsweise das DMS prüft Berechtigung und erwartete Basisversion beim Schreiben atomar. Hat sich der gemeinsame Stand inzwischen geändert, entsteht kein stilles Überschreiben: Die eigene Fassung bleibt erhalten und es folgt eine bewusste Konfliktentscheidung. Falls ein Anbieter diese Prüfung nicht unterstützt, darf der Connector keine gleichwertige Konfliktsicherheit behaupten. Lokale Autosicherung, gemeinsame Veröffentlichung und Aktualisierung eines geöffneten Editors sind unterschiedliche Vorgänge; ob „Speichern“ zugleich veröffentlicht, ist noch festzulegen. Es soll nicht ohne Bedarf ein zusätzlicher Freigabeklick für jede normale Änderung entstehen.

### 6.3 Verhalten bei Konflikten

| Situation | Empfohlenes Verhalten |
|---|---|
| Zwei Personen bearbeiten verschiedene Dokumente | Beide arbeiten unabhängig; neue Versionen werden synchronisiert. |
| Zwei Personen öffnen dasselbe Dokument | Beide dürfen lesen; allein durch das Öffnen entsteht keine Sperre. |
| Zwei Personen beginnen gleichzeitig zu ändern | Aktivitätsanzeige ist nur Beratung. Versionierte eigene Fassungen und die atomare Prüfung bei Veröffentlichung verhindern verlorene Änderungen. |
| Eine Person bearbeitet offline, eine andere ändert online dieselbe DOCX | Beide Fassungen erhalten; gemeinsame Basis und Unterschiede zeigen; neue Version nach bewusster Entscheidung. |
| Eine neuere Version trifft ein, während das Dokument offen ist | Hinweis und bewusste Aktualisierung; aktive Ansicht und eigener Entwurf bleiben erhalten. |
| Dokument bewusst exklusiv reserviert | Server/DMS bestätigt Reservierung samt Owner, Basisversion und Regeln. Andere sehen die Reservierung; Offline-Arbeit setzt sie nicht generell voraus. |
| Lease ist abgelaufen oder wurde nach einem Geräteausfall aufgehoben | Offline-Arbeit bleibt Entwurf. Veröffentlichung prüft erneut Rechte, Sperrgeneration und Basisversion. |
| Zugriff wird während des Flugs entzogen | Kein sofortiger technischer Rückruf eines vollständig getrennten Geräts. Beim nächsten Kontakt sperren; eine zuvor festgelegte Offline-Berechtigungsdauer begrenzt die Nutzbarkeit innerhalb der verwalteten App. |
| Zugriff wurde entzogen, ungesendete Arbeit liegt vor | Geschützte Quarantäne und Übergabe an berechtigte Kanzleiverantwortliche; kein automatisches „als privates Projekt behalten“. |

Eine Reservierung kann nur bei erreichbarer maßgeblicher Instanz verbindlich erworben werden. Offline lässt sich ein Entwurf anlegen, aber keine neue globale Exklusivität behaupten. Ablauf oder Aufhebung der Reservierung darf niemals still einen neueren Stand überschreiben. Bei unterstütztem Microsoft-/DMS-Co-authoring verwendet Legal Work diesen Modus und startet daneben keine konkurrierende exklusive Bearbeitung.

### 6.4 Zwei Betriebsarten für Dateien

**Native Legal-Work-Mandate:** Eigenwelt oder ein entsprechend unterstützter Kanzleiserver verwaltet Rechte, unveränderliche Versionen und Sync. Geräte halten erlaubte Repliken. Der bestehende Transport und seine Konfliktbehandlung können dafür weiterverwendet werden.

**Mandate mit führendem DMS:** Legal Work hält stabile Referenzen, erlaubte Metadaten und kontrollierte Arbeitskopien. Dauerhafte Dokument-ID, Repository/Library, Version und Mandats-/Mandantenkennung bleiben erhalten. Speichern als neuer Entwurf, als neue DMS-Version und als eigenständiges Dokument müssen unterscheidbare Vorgänge sein. Ein umbenannter Ordner darf keine neue Akte erzeugen.

DMS-Dokumente werden standardmäßig nicht nochmals unkontrolliert in ein breiter freigegebenes Legal-Work-Projekt kopiert. Der Connector prüft die Rechte der handelnden Person. Eine Anmeldung über ein allgemein berechtigtes Sammelkonto darf persönliche Informationsbarrieren nicht aufheben. Bestehende Legal-Work-Teamkonten sind praktisch, setzen innerhalb der Anwendung aber engere Regeln als ein außerhalb verwendetes Provider-Token; sie sind daher kein gleichwertiger Ersatz für benutzerbezogene DMS-Authentifizierung. [Team-Speichergrenzen](https://github.com/eigenweltlabs/legalwork/blob/bb77ce7ce2ef0a665ad3f0704a7994b32b54672d/docs/team-storage.md).

Für jeden angebundenen Anbieter braucht es einen Fähigkeitsvertrag: Identitäten auflösen, Berechtigungen prüfen, Versionen lesen/erzeugen, reservieren/freigeben, Änderungen erkennen und Offline-Kopien verwalten. Nicht unterstützte Funktionen werden im Produkt kenntlich gemacht.

### 6.5 Was eine verwaltete Offline-Kopie leisten kann

Für Kanzleidaten empfehle ich einen verwalteten, verschlüsselten Cache mit gerätebezogenen Schlüsseln, festgelegter Offline-Dauer und nachvollziehbarer Lösch-/Sperrbestätigung beim Wiederverbinden. Downloads/Exporte in normale Ordner sind eine separate Berechtigung. Entwürfe nach Rechteentzug werden geschützt aufbewahrt, bis ein berechtigter Verantwortlicher sie übernimmt.

Das ist eine Architekturentscheidung, kein Versprechen, bereits exportierte Klartextkopien oder Screenshots zurückholen zu können. Gegen Umgehungen durch den Gerätebesitzer braucht es zusätzliche Gerätekontrolle; eine UI-Sperre allein kann das nicht leisten. Die bestehende freie lokale Arbeitsweise kann für persönliche Projekte bestehen bleiben. Kanzleiverwaltete Daten dürfen ihre Regeln beim Abmelden jedoch nicht einfach verlieren.

## 7. Zeiterfassung und spätere Kanzleifunktionen vorbereiten

Abrechenbare Zeiterfassung sollte an das **Mandat**, die leistende Person und gegebenenfalls Projekt und konkreten Arbeitsvorgang gebunden sein. Interne Zeiten können einem internen Projekt ohne Mandant zugeordnet werden und benötigen keinen künstlichen Kunden. Mandant, Rechnungsempfänger und Billing Partner sind getrennte Referenzen. Eine Leistung benötigt mindestens Datum, Dauer/Menge, Tätigkeit, Beschreibung, Verrechenbarkeit, Status und Herkunft. Sätze und Tarifregeln brauchen eigene Versionen beziehungsweise den bei Freigabe festgehaltenen Stand.

Sinnvoller Ablauf:

**Entwurf → eingereicht → geprüft/freigegeben → abgerechnet → gegebenenfalls nachvollziehbar korrigiert.**

Ein Associate kann eigene Zeiten erfassen, ohne fremde Honorarsätze zu sehen. Billing kann Leistungen aufbereiten; der zuständige Partner genehmigt den vorgesehenen Umfang. Bereits abgerechnete Leistungen werden nicht unbemerkt überschrieben. Für Finanzdaten ist „die letzte Änderung gewinnt“ kein ausreichendes Sync-Verfahren.

Offline-Erfassung erzeugt lokale, eindeutig identifizierte Ereignisse. Beim Wiederverbinden werden sie genau einmal übernommen. Änderungen an bereits eingereichten oder freigegebenen Positionen lösen eine kontrollierte Revision aus. Mehrere Geräte dürfen nicht dieselben Minuten doppelt abrechnen. KI kann aus Arbeitsschritten Leistungsvorschläge erzeugen, aber Laufzeit eines Agenten ist nicht automatisch abrechenbare Anwaltszeit.

Weitere anschlussfähige Bereiche sind Mandatsannahme und Kollisionsprüfung, Kontakte/Beteiligte, Budgets, Fristen mit Vier-Augen-Freigabe, Korrespondenzzuordnung, Mandantenportale, Signaturen, Archivierung und Aufbewahrung. Bestehende Buchhaltung, ERV und Honorarlogik würde ich zunächst anbinden. Österreichische Abrechnung und Gerichtskommunikation sind eigenständige Fachprodukte; sie sollten nicht beiläufig als Teil des ersten Organisationsumbaus entstehen.

## 8. Umsetzung ohne unnötigen Neubau

### Schritt 1: Mandanten und Mandate als echte Daten

Die vorhandene Organisation wiederverwenden. Mandanten mit stabilen IDs und optionale Projektzuordnungen hinzufügen; mehrere Projekte pro Mandant und Projekte ohne Mandant gehören zum ersten Umfang. Die Sidebar kann zwischen Projektansicht und Gruppierung nach Mandanten wechseln. Mandatsidentitäten und Beteiligte ergänzen, wo mehrere Projekte dieselbe Akte teilen oder DMS-/Finanzbezüge sie benötigen; die Oberfläche verlangt keine zusätzliche Anlage für den einfachen Einprojektfall. Vorhandene Aktenmetadaten bleiben als anpassbare Zusatzfelder erhalten; Identitäten werden nicht weiter als beliebige Freitextfelder geführt.

Für Bestandsprojekte schlägt ein Assistent die Zuordnung vor: „Mandat“, „Arbeitsbereich eines vorhandenen Mandats“, „internes Projekt“ oder „noch nicht zugeordnet“. Gleiche Mandantennamen werden nicht ungeprüft zusammengeführt. Bestehende IDs, Pfade, Chats und Freigaben bleiben erhalten, bis eine explizite Migration erfolgt.

### Schritt 2: Rechte und Kanzleibesitz verbindlich machen

Teamrollen und Rechte für Lesen, Bearbeiten, Teamverwaltung, Teilen, Offline-Kopien und Finanzdaten einführen. Schutzregeln serverseitig auf allen Zugriffswegen prüfen. Kanzleimandate gehören fachlich der Organisation, mit übertragbarer Verantwortung; das Ausscheiden des Erstellers darf sie nicht verwaisen lassen. Eine rein visuelle Mandantenübersicht kann vorher erscheinen, darf aber noch keine neue Sicherheitsgrenze versprechen.

### Schritt 3: Dokumentidentität und Offline-Regeln

Dokument-IDs, unveränderliche Versionen, Herkunft und Basisversionen ergänzen. Zuverlässige lokale Entwürfe, automatische Offline-Fortsetzung bereits verfügbarer Arbeit, optionale Offline-Vorhaltung und sichtbare Verbindungs-/Versionszustände umsetzen. Offene Ansichten dürfen nicht automatisch sperren; eigene schreibende Fenster werden koordiniert, teamweite Aktivität wird angezeigt. Neue gemeinsame Versionen ersetzen aktive Ansichten nicht still. Veröffentlichungen prüfen die Basisversion atomar. Ausdrückliche Reservierungen und geschützte Konflikt-/Entwurfswiederherstellung ergänzen. Der erste unterstützte DMS-Pfad muss einschließlich Rechteentzug und Save-back vollständig geprüft sein. Die vorhandenen generischen Dateiadapter bleiben als einfachere Integrationen bestehen; der bestehende Dateisync allein erfüllt diese zusätzlichen Versions- und Editorverträge noch nicht.

### Schritt 4: Leistungserfassung und Finanzintegration

Zunächst lokale/offline Zeiterfassung und kontrollierte Übertragung in das führende Kanzleisystem. Danach mandatsbezogene Freigaben, Budgets und Auswertungen. Erst anhand konkreter Kanzleibedarfe entscheiden, welche Abrechnungsfunktionen Legal Work selbst führen soll.

### Schritt 5: Gemeinsames Bearbeiten und Cloud-Ausführung erweitern

Co-authoring, externe Portale und autonome Cloud-Aufträge auf dieselben Identitäten und Rechte setzen. Vor jeder Cloud-Ausführung erneut prüfen, ob die Person, das Mandat und die ausgewählten Daten dafür zugelassen sind. Ein persönlicher Cloud-Assistent wird dadurch nicht zu einem zweiten, unabhängigen Berechtigungssystem.

**Für die Umsetzung besonders geeignete vorhandene Ansatzpunkte:** `packages/types/src/workspace.ts` und die Projektverträge für stabile Zuordnungen; `project-sync.ts`/`project-file-sync.ts` für Replikation; `file-storage/common.ts` für explizite Anbieterfähigkeiten; Kanzleirichtlinien für zentral vorgegebene Regeln; die vorhandenen Kalender-/Aufgabensysteme als Mandatsbestandteile. Die serverseitigen Änderungen müssen parallel im nicht zugänglichen `model-api` erfolgen. Das ist keine reine Sidebar-Änderung.

## 9. Woran die erste belastbare Version zu messen ist

1. **Mandantenübersicht und Navigation:** Mehrere Projekte desselben Mandanten erscheinen gemeinsam oder in einer flachen Projektansicht, je nach persönlicher Auswahl. Sie behalten getrennte Teams; ein gemeinsamer Mandant erweitert den Zugriff nicht. Interne Projekte funktionieren mit leerer Mandantenzuordnung, auch als geteilte Kanzleiprojekte.
2. **Billing:** Eine Abrechnungsmitarbeiterin kann eine zugewiesene Rechnung vorbereiten und notwendige Leistungen lesen, ohne Strategieunterlagen, Chats oder DMS-Dokumente zu erhalten.
3. **Informationsbarriere:** Eine gesperrte Person erhält auch über Suche, Agenten, Wiki-Ableitungen, Vorschauen, Benachrichtigungen und Export keinen unzulässigen Zugriff.
4. **Verbindungsverlust:** Bereits vollständig lokal verfügbare und erlaubte Dokumente bleiben ohne besonderen Flugzeugablauf nutzbar. Änderungen bleiben lokal erhalten. Optionale Offline-Vorhaltung zeigt noch ausstehende Downloads; fehlende Inhalte und Cloud-Funktionen sind klar erkennbar.
5. **Viele Fenster und konkurrierende Bearbeitung:** 17 offene Arbeitsbereiche sperren keine unberührten Dokumente. Eigene Fenster verlieren bei der Bearbeitungsübergabe keinen Entwurf. Eine eingehende Version ersetzt keine aktive Ansicht. Gleichzeitige und Offline-DOCX-Änderungen bleiben erhalten; veraltete Aktivitätsanzeigen oder Reservierungen können weder eine Dauersperre noch ein stilles Überschreiben auslösen.
6. **Rechteentzug:** Ein online verbundenes Gerät stoppt weitere Zugriffe; offline gelten die vorab definierten Grenzen. Ungesendete Kanzleiarbeit wird geschützt übernommen und nicht privatisiert.
7. **DMS-Anbindung:** Umbenennen/Verschieben erhält Identität und Quellenbezug. Save-back erzeugt die erwartete Version im richtigen DMS und respektiert dessen Rechte.
8. **Ausscheiden eines Partners:** Mandate, Versionen, Aufgaben und Freigabeverantwortung bleiben bei der Kanzlei und können kontrolliert übernommen werden.
9. **Zeiterfassung:** Wiederholter Sync erzeugt keine doppelten Leistungen; freigegebene oder abgerechnete Positionen ändern sich nur über den vorgesehenen Prozess.

## 10. Noch zu entscheidende Produktfragen

1. **Bedeutung von Speichern:** Veröffentlicht normales Speichern eine neue Teamversion, während Autosave nur den eigenen Entwurf schützt, oder werden gemeinsame Versionen nach einer anderen klaren Regel erzeugt? Eine zusätzliche ausdrückliche Freigabe ist nicht für jede Routineänderung automatisch nötig.
2. **Parallele Bearbeitung:** Reicht standardmäßig ein Hinweis mit bewusstem Fortsetzen als eigene Fassung, oder sollen bestimmte Dokumente verbindlich reserviert werden? Unveröffentlichte Entwürfe allein dürfen nicht unbegrenzt blockieren.
3. **Sichtbare Mandatsebene:** Genügt zunächst Mandant → mehrere Projekte, oder brauchen konkrete Kanzleien bereits Mandat → mehrere Projekte? Die fachlichen Identitäten müssen die Erweiterung erlauben, ohne jede Nutzung mit einer weiteren Ebene zu belasten.

Die vorgeschlagene Richtung ist eine lokale juristische Arbeitsumgebung mit stabilen Mandanten-/Mandatsbezügen, durchgängigen Rechten und einem verlässlichen Dokumentlebenszyklus. Die persönliche Navigation und die Zahl offener Arbeitsbereiche dürfen diese fachlichen Beziehungen weder verändern noch unnötige Sperren erzeugen.
