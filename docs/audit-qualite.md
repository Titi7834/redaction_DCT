# Audit qualité DCT — Groupe SupEvents (Tom LEPRIEUR, Arthur L'AFFETER, Tiago DA COSTA)

**Audité par** : auto-audit (binôme croisé indisponible — posture d'auditeur externe adoptée)
**Date** : 2026-04-30
**Version DCT** : `main` après TP 1.6

Périmètre audité : `06.4-vue-donnees.md`, `07-modules.md`, `08-interfaces-api.md`, `09-exigences-transverses.md`, `10-tracabilite.md`, `docs/adr/*`. Notation 0 / 1 / 2 par critère. Une note sans justification est exclue.

---

## Catégorie 1 — Structure (12 points / 6 critères)

| N°  | Critère                                                       | Score | Commentaire                                                                                       | Action recommandée                                                                          |
|-----|---------------------------------------------------------------|-------|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| S1  | Page de garde présente (titre, version, équipe, contexte)      | **0** | Aucun fichier `00-page-garde.md`. Le lecteur arrive directement sur une section technique.        | Créer `00-page-garde.md` avec titre projet, version, équipe, date, contexte CDC.            |
| S2  | Sommaire / table des matières navigable                        | **0** | Aucune TOC à la racine. Lecteur doit deviner l'ordre des fichiers via leur préfixe numérique.     | Ajouter une TOC dans `00-page-garde.md` avec liens markdown vers chaque section.            |
| S3  | Numérotation cohérente des sections                            | **1** | Numérotation respectée (§ 6.4, § 7, § 8, § 9, § 10) mais sections § 6.1, § 6.2, § 6.3 manquantes.  | Soit produire un stub `06.1-vue-architecture.md` soit signaler explicitement le report.    |
| S4  | Glossaire ≥ 10 termes définis                                  | **0** | Aucun glossaire. Termes spécifiques (PCI-DSS, OIDC, JWT, HMAC, STRIDE, SLO, SLI, 3DS2, PKCE, SSO, ADR, DLQ, outbox, idempotence, jauge) non définis. | Créer `00-glossaire.md` avec ≥ 15 entrées.                                                  |
| S5  | Historique des révisions / changelog                           | **0** | Aucun historique. Impossible de savoir ce qui a évolué depuis la V0.                              | Ajouter `CHANGELOG.md` à la racine, format Keep a Changelog.                                |
| S6  | Convention de nommage des fichiers cohérente                    | **2** | Tous les fichiers en kebab-case + préfixe numérique (`06.4-`, `07-`, `08-`...).                    | RAS.                                                                                        |

**Total Catégorie 1 : 3 / 12**

---

## Catégorie 2 — Complétude (14 points / 7 critères)

| N°  | Critère                                                       | Score | Commentaire                                                                                       | Action recommandée                                                                          |
|-----|---------------------------------------------------------------|-------|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| C1  | Vue d'architecture globale (§ 6.1 à 6.3)                       | **0** | Sauté directement à § 6.4. Aucune vue logique / processus / déploiement.                          | Produire un stub minimal § 6.1 vue logique avant TP 1.8.                                    |
| C2  | Modèle de données complet (§ 6.4)                              | **2** | ER diagramme 9 entités, dictionnaire complet, contraintes, index, partitionnement.                 | RAS.                                                                                        |
| C3  | Modules détaillés (§ 7) sur tous les modules majeurs           | **1** | 3 fiches sur 6 modules (`AuthModule`, `TicketModule`, `NotificationModule`). `EventModule`, `PaymentModule`, `UserModule` listés stubs. | Produire les 3 fiches manquantes en TP 1.8.                                                 |
| C4  | Contrats API (§ 8) — endpoints + erreurs + événements          | **2** | 22 endpoints, RFC 7807, 2 événements documentés avec schéma JSON.                                  | RAS sauf ajout `ticket.cancelled` (cf. CO3).                                                |
| C5  | Exigences transverses (§ 9) — sécurité, perf, RGPD             | **2** | STRIDE 2 flux, 3 SLO chiffrés + budget erreur, registre RGPD + 3 procédures.                        | RAS.                                                                                        |
| C6  | Traçabilité (§ 10) — matrice exigences × DCT                   | **2** | 13 EF + 8 ENF référencées, synthèse honnête, lecture par module.                                   | RAS.                                                                                        |
| C7  | ADR ≥ 3 + index                                                | **2** | ADR-001, ADR-002, ADR-003 + README index. Format Nygard respecté.                                  | RAS.                                                                                        |

**Total Catégorie 2 : 11 / 14**

---

## Catégorie 3 — Cohérence (12 points / 6 critères)

| N°  | Critère                                                       | Score | Commentaire                                                                                       | Action recommandée                                                                          |
|-----|---------------------------------------------------------------|-------|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| CO1 | Nommage entités cohérent § 6.4 ↔ § 7 ↔ § 8                     | **2** | `Ticket`, `Event`, `Payment`, `User` partout sans synonymes parasites.                             | RAS.                                                                                        |
| CO2 | Codes HTTP cohérents § 7 ↔ § 8                                 | **2** | Codes 409, 422, 502, 401, 403 présents dans les deux sections, alignés.                            | RAS.                                                                                        |
| CO3 | Événements documentés en § 7 figurent en § 8                   | **1** | `ticket.cancelled` cité 4 fois en § 7.2 / § 7.3 mais absent du tableau § 8 (seuls `ticket.confirmed` et `payment.failed` détaillés). | Ajouter fiche `ticket.cancelled` à la § 8.                                                  |
| CO4 | Modules cités en § 10 existent en § 7                          | **1** | `EventModule`, `PaymentModule`, `UserModule` cités sur 12 lignes de matrice mais flaggés *(à détailler)* — non encore produits. | Compléter en TP 1.8.                                                                        |
| CO5 | ADR cités existent dans `/docs/adr/`                           | **2** | ADR-001, ADR-002, ADR-003 référencés en § 7.1, § 7.2, § 9.1.1 — tous présents.                     | RAS.                                                                                        |
| CO6 | Sections DCT citées existent                                   | **1** | Liens vers § 6.1 (depuis § 7), § 9.4 (depuis § 9.3), § 10 (stack obs.), § 11 (qualité doc) présents mais sections absentes. | Soit produire les sections, soit remplacer par mention explicite « à produire en TP 1.8 ». |

**Total Catégorie 3 : 9 / 12**

---

## Catégorie 4 — Lisibilité (12 points / 6 critères)

| N°  | Critère                                                       | Score | Commentaire                                                                                       | Action recommandée                                                                          |
|-----|---------------------------------------------------------------|-------|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| L1  | Tableaux structurés majoritaires sur listes denses             | **2** | Dictionnaires de données, codes erreur, matrice traçabilité, STRIDE, SLO — tout en tableaux.       | RAS.                                                                                        |
| L2  | Phrases ≤ 25 mots dans le corps narratif                       | **1** | § 6.4 « Stratégie de stockage » : phrase de 95+ mots. § 9.1.1 paragraphe : 2 phrases > 30 mots.    | Découper les phrases > 30 mots dans `06.4-vue-donnees.md` (paragraphe stratégie stockage).  |
| L3  | Diagrammes Mermaid rendus sans erreur                          | **2** | erDiagram, stateDiagram-v2, flowchart, sequenceDiagram tous syntaxiquement valides (vérif. mermaid.live). | RAS.                                                                                        |
| L4  | Mots-clés en gras pour faciliter le scan                        | **1** | Très dense en § 9 (good), peu en § 6.4 et § 7.2 (paragraphes en bloc).                              | Ajouter du gras sur les termes structurants en § 6.4 stratégie stockage et § 7.2 algo.      |
| L5  | Titres décisionnels (affirmation) plutôt qu'interrogatifs       | **2** | ADR titres affirmatifs (« Adopter X »), sections DCT titres descriptifs cohérents.                  | RAS.                                                                                        |
| L6  | Voix active dominante dans les paragraphes critiques            | **1** | Style technique français → beaucoup de passives (« est stocké », « est délégué »). Acceptable mais pourrait s'alléger. | Réécrire les passages clés en voix active (§ 9.1.1 paragraphe descriptif).                  |

**Total Catégorie 4 : 9 / 12**

---

## Catégorie 5 — Maintenabilité (10 points / 5 critères)

| N°  | Critère                                                       | Score | Commentaire                                                                                       | Action recommandée                                                                          |
|-----|---------------------------------------------------------------|-------|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| M1  | Métadonnée « date de dernière mise à jour » par fichier        | **0** | Aucune date dans les fichiers (sauf ADR qui ont leur champ Date). Pas de front-matter.            | Ajouter en pied de chaque `.md` une mention `*Dernière mise à jour : YYYY-MM-DD*`.          |
| M2  | Fichier CODEOWNERS                                             | **0** | Absent. Aucune attribution de responsabilité par section.                                          | Créer `.github/CODEOWNERS` (Phase C).                                                       |
| M3  | Pipeline CI de validation des `.md`                            | **0** | Aucun workflow. Aucun lint, aucune vérif liens, aucun rendu Mermaid auto.                          | Créer `.github/workflows/docs-quality.yml` (Phase C).                                       |
| M4  | Liens internes valides                                         | **1** | Renvois vers § 6.1, § 9.4, § 10 (stack obs.), § 11 sont des liens conceptuels morts (sections absentes). | Corriger les renvois ou produire les sections.                                              |
| M5  | Index ADR à jour                                               | **2** | `docs/adr/README.md` liste les 3 ADR avec statut, date, lien.                                      | RAS sauf : recréer `docs/adr/template.md` qui a été supprimé.                              |

**Total Catégorie 5 : 3 / 10**

---

## Synthèse

**Total général** : **35 / 60 (58 %)**

| Catégorie       | Score    | Note relative |
|-----------------|----------|---------------|
| Structure       | 3 / 12   | Insuffisant   |
| Complétude      | 11 / 14  | Bon           |
| Cohérence       | 9 / 12   | Correct       |
| Lisibilité      | 9 / 12   | Correct       |
| Maintenabilité  | 3 / 10   | Insuffisant   |

Lecture : la DCT est solide sur le **contenu technique** (Complétude, Cohérence, Lisibilité ≥ 75 %) mais faible sur les **fondations documentaires** (Structure, Maintenabilité ≤ 30 %). Profil typique d'une DCT étudiante construite TP par TP.

### Top 5 des défauts les plus impactants

| Rang | Critère(s)        | Description                                                                                       | Impact estimé                                                                                       |
|------|-------------------|---------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| 1    | S1, S2            | Page de garde + sommaire absents.                                                                 | Lecteur externe ouvre `06.4-vue-donnees.md` sans contexte projet → DCT perçue comme un fragment.    |
| 2    | S4                | Glossaire absent. ≥ 15 termes techniques non définis (PCI-DSS, OIDC, STRIDE, SLO/SLI, 3DS2...).    | Auditeur non-tech et nouvel arrivant doivent googler en parallèle de la lecture.                    |
| 3    | M2, M3            | CODEOWNERS et CI absents.                                                                          | Aucune garantie qu'une PR future ne casse les liens, le rendu Mermaid ou l'orthographe.             |
| 4    | C1, CO6           | Sections § 6.1 à § 6.3 (vue logique / processus / déploiement) absentes alors que référencées en § 7. | Lecteur n'a pas la vue d'ensemble avant de plonger dans les modules.                                |
| 5    | M1, S5            | Aucune métadonnée temporelle (date par fichier, changelog).                                        | Document non auditable temporellement — impossible de savoir si une section est à jour ou périmée.  |

### Plan de correction (priorité × impact)

| # | Action                                                                                       | Impact | Coût  | Décision Phase B                          |
|---|----------------------------------------------------------------------------------------------|--------|-------|-------------------------------------------|
| 1 | Créer `00-page-garde.md` avec titre + équipe + contexte + TOC                                | Élevé  | Faible| **À corriger en Phase B** (top 1).        |
| 2 | Créer `00-glossaire.md` avec ≥ 15 termes                                                     | Élevé  | Faible| **À corriger en Phase B** (top 2).        |
| 3 | Créer `CHANGELOG.md` + ajouter pied de page « Dernière mise à jour » sur tous les `.md`      | Élevé  | Faible| **À corriger en Phase B** (top 3).        |
| 4 | Recréer `docs/adr/template.md` (supprimé)                                                    | Moyen  | Faible| **À corriger en Phase B** (top 4).        |
| 5 | Ajouter événement `ticket.cancelled` à la § 8                                                | Moyen  | Faible| **À corriger en Phase B** (top 5).        |
| 6 | Découper phrases > 30 mots § 6.4 stratégie stockage                                          | Moyen  | Moyen | À traiter en TP 1.8.                      |
| 7 | Produire stubs § 6.1 / § 6.2 / § 6.3 + fiches manquantes § 7.4 / § 7.5 / § 7.6              | Élevé  | Élevé | **TP 1.8** (sprint final).                |
| 8 | Produire sections § 11 qualité doc et § 9.4 accessibilité                                    | Moyen  | Élevé | **TP 1.8**.                                |
| 9 | Voix active sur § 9.1.1                                                                       | Faible | Moyen | À ne pas traiter — ratio coût / valeur défavorable. |

### Phase C (gouvernance + CI)

- **CODEOWNERS** : à produire — 1 owner par section principale, fallback collectif (cf. fichier `.github/CODEOWNERS`).
- **CI workflow** : à produire — markdownlint + markdown-link-check + mermaid-cli (cf. `.github/workflows/docs-quality.yml`).
- **Validation** : PR de test à ouvrir + capture dans `ci-validation.md`.

---

*Dernière mise à jour : 2026-04-30*
