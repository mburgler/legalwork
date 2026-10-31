# Law Firm Infrastructure — Notizen

**Archiviert:** 31. Oktober 2026. **Inhaltlicher Planstand:** 23. Oktober 2026.

Diese Sammlung hält die Recherche, die Produktdiskussion, den adversarial Review und den daraus überarbeiteten Plan für Legal Work fest. Sie beschreibt ein vorgeschlagenes Zielsystem. Historische PR- und Herstellerbefunde wurden für dieses Archiv nicht erneut vollständig überprüft.

## Einstieg

| Dokument | Inhalt |
|---|---|
| [Kompakte Zusammenfassung](summary.md) | Alle wesentlichen Inhalte: Fachmodell, Rechte, Speichern, Sync, Zusammenarbeit, Offline-Arbeit, Versionen, Umsetzung und offene Entscheidungen. |
| [Diskussion und Entscheidungen](discussion-notes.md) | Ausgangsfrage, Nutzeranforderungen, Präzisierungen und die Übernahme beziehungsweise Ablehnung von Review-Empfehlungen. |
| [Proposal](../../proposal/client-matter-collaboration.md) | Die bereits hochgeladene, eigenständig lesbare Planfassung mit Umsetzungsetappen und Abnahmeszenarien. |
| [Vollständiger Bericht](Bericht.md) | Recherche zu Legal Work, Pandect/Legal IDE/Project Temple und etablierten Kanzleisystemen; überarbeitetes Fach- und Architekturmodell; Review-Entscheidungen. |
| [Fable-Review](Fable-Adversarial-Review.md) | Unveränderte Ausgabe des unabhängigen Reviews durch Claude Code / Fable 5.1. |
| [Geprüfte frühere Planfassung](fable-review-plan-snapshot.md) | Eingefrorene Eingabe des Reviews. Die darin verwendeten Zeilenverweise beziehen sich auf diese Fassung. |
| [GitHub-Recherche-Snapshot](github-snapshot.json) | Metadaten der damals betrachteten 200 zuletzt aktualisierten PRs und 81 Branches sowie die geprüften Commit-IDs. |
| [Dateimanifest](manifest.json) | Herkunft und Prüfsummen der archivierten Dokumente. |

## Bezug zum Draft-PR

- [PR #328: docs: Law Firm Infrastructure (Matters and Clients, DMS, Roles and Collaboration)](https://github.com/eigenweltlabs/legalwork/pull/328).
- Grundlage dieses Notizbranches ist der veröffentlichte Proposal-Commit `173511635e4df566563e295693d83d15d5b64ee8` aus `mburgler/legalwork`.
- Die Proposal-Datei dieses Commits bleibt enthalten. Die zusätzlichen Notizen liegen vollständig unter `docs/notes/law-firm-infrastructure/`.

## Wie die Stände zu lesen sind

Für die aktuelle Empfehlung innerhalb dieser Sammlung gelten die Proposal-Datei und der überarbeitete Bericht. Der Fable-Review beurteilt eine frühere Fassung; nicht jede seiner Gegenempfehlungen wurde übernommen. Die Auswahl ist in den Diskussionsnotizen und in Abschnitt 11 des Berichts dokumentiert.

Die ursprüngliche Recherche fixiert ihren Entwicklungsstand mit Commit-IDs. Ein damaliger Merge belegt kein heutiges Deployment. Das begleitende Backend `model-api` war bei der Recherche nicht zugänglich; sein tatsächlicher Vertrag bleibt vor einer Implementierung zu prüfen. Es wurden für diese Archivierung keine Produktfunktionen implementiert oder Laufzeittests ausgeführt.
