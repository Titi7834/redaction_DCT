# Architecture Decision Records — SupEvents

Ce dossier rassemble les ADR (Architecture Decision Records) au format Nygard documentant les décisions structurantes du projet SupEvents. Les ADR ne sont jamais supprimés : leur statut évolue (`Proposé` → `Accepté` → `Déprécié` / `Remplacé par ADR-XXX`).

## Index des ADR

| Numéro    | Titre                                                                                                    | Statut   | Date       | Auteurs                                              |
|-----------|----------------------------------------------------------------------------------------------------------|----------|------------|------------------------------------------------------|
| [ADR-001](ADR-001.md) | Adopter un verrou pessimiste PostgreSQL pour l'allocation d'une place sur événement              | Accepté  | 2026-04-29 | Tom LEPRIEUR, Arthur L'AFFETER, Tiago DA COSTA       |
| [ADR-002](ADR-002.md) | Adopter une idempotence par clé client (`Idempotency-Key`) pour les opérations d'écriture sensibles | Accepté  | 2026-04-29 | Tom LEPRIEUR, Arthur L'AFFETER, Tiago DA COSTA       |
| [ADR-003](ADR-003.md) | Adopter une authentification JWT stateless courte + refresh token rotatif plutôt que des sessions serveur via Redis | Accepté  | 2026-04-29 | Tom LEPRIEUR, Arthur L'AFFETER, Tiago DA COSTA       |

## Convention d'usage

- **Numérotation** : séquentielle, jamais réutilisée (`ADR-001`, `ADR-002`, ...).
- **Format** : voir [`template.md`](template.md). Six rubriques obligatoires : Contexte, Décision, Alternatives envisagées (≥ 2), Conséquences (positives + négatives + dette acceptée), Liens DCT.
- **Workflow** : tout ADR est ouvert en pull request, revu par au moins un autre rédacteur ou tech lead avant passage au statut `Accepté`.
- **Liens croisés** : chaque ADR cite explicitement les sections de la DCT (§ 6.4, § 7, § 8, § 9...) qu'il influence ; les sections concernées de la DCT mentionnent l'ADR pertinent.
