## §10 — Matrice de traçabilité

##### Fait par Tom LEPRIEUR, Arthur L'AFFETER et Tiago DA COSTA

Cette section croise les exigences du CDC SupEvents (13 EF + 8 ENF, hypothèse de référencement V1 — la numérotation et l'intitulé exacts sont à aligner sur le CDC officiel) avec les éléments de la DCT produits dans les TP 1.4, 1.5 et 1.6 : modules de la § 7, sections § 6.4 (données), § 8 (API), § 9 (transverses) et ADR du dossier `/docs/adr/`. La matrice se lit dans deux directions : par exigence (vérifier qu'elle est couverte) et par module (vérifier qu'aucun composant n'est orphelin). Les exigences explicitement non couvertes à ce stade sont signalées dans la colonne ADR avec une note de renvoi vers le TP / sprint qui les traitera.

### §10.1 — Matrice principale

| Réf.   | Intitulé (court)                          | Module(s)                                   | Section(s) DCT                                           | ADR                                  |
|--------|-------------------------------------------|---------------------------------------------|----------------------------------------------------------|--------------------------------------|
| EF-01  | Authentification SSO école (OIDC)         | `AuthModule`                                | § 6.4 (`User`, `RefreshToken`), § 8 (`/auth/*`)          | ADR-003                              |
| EF-02  | Création / édition d'événement            | `EventModule` *(à détailler en § 7)*        | § 6.4 (`Event`, `Category`), § 8 (`POST /events`, `PATCH`) | —                                    |
| EF-03  | Publication / annulation d'événement      | `EventModule`                               | § 6.4 (`Event.status`), § 8 (`/events/{id}/publish`, `/cancel`), § 9.1.1 | —                                    |
| EF-04  | Catalogue public d'événements             | `EventModule`                               | § 6.4 (`Event.status='published'`, index dédié), § 8 (`GET /events`) | —                                    |
| EF-05  | Recherche et filtrage d'événements        | `EventModule`                               | § 6.4 (index GIN tsvector), § 8 (`GET /events/search`)   | —                                    |
| EF-06  | Inscription à un événement gratuit        | `TicketModule`                              | § 6.4 (`Ticket`), § 7.2, § 8 (`POST /events/{id}/tickets`), § 9.1.1 | ADR-001, ADR-002                     |
| EF-07  | Inscription à un événement payant (Stripe)| `TicketModule`, `PaymentModule` *(à détailler)* | § 6.4 (`Ticket`, `Payment`), § 7.2, § 8 (`/payment-intent`, `/webhook`), § 9.1.1, § 9.1.2 | ADR-001, ADR-002                     |
| EF-08  | Annulation d'un billet par l'utilisateur  | `TicketModule`, `PaymentModule`             | § 6.4 (`Ticket.status`), § 7.2, § 8 (`DELETE /tickets/{id}`) | —                                    |
| EF-09  | Tableau de bord organisateur (KPI + participants) | `EventModule`, `TicketModule`         | § 8 (`/organizer/events/.../participants`, `/kpis`)      | —                                    |
| EF-10  | Export CSV des participants               | `EventModule`, `TicketModule`               | § 6.4 (S3 convention `exports/...`), § 8 (`/participants.csv`) | —                                    |
| EF-11  | Notifications utilisateur (e-mail)        | `NotificationModule`                        | § 6.4 (`Notification`), § 7.3, § 8 (événements `ticket.confirmed`, `payment.failed`) | —                                    |
| EF-12  | Validation / révocation administrative des organisateurs | `UserModule` *(à détailler)*    | § 6.4 (`Organizer.status`), § 8 (`/admin/organizers/.../validate`, `/revoke`) | —                                    |
| EF-13  | Liste d'attente sur événement complet     | `TicketModule`                              | § 6.4 (`WaitingListEntry`), § 7.2 (cas limite `EVENT_FULL_QUEUED`) | —                                    |
| ENF-01 | Performance — latence p95 ≤ 500 ms, 500 utilisateurs concurrents | `TicketModule`, `EventModule` | § 6.4 (index stratégiques), § 9.2 (ENF-PERF-01, ENF-PERF-03) | ADR-001                              |
| ENF-02 | Disponibilité ≥ 99,5 % (budget 3 h 36 min/mois) | Tous les modules backend            | § 9.2 (ENF-PERF-02), § 8 (codes 502/503)                 | —                                    |
| ENF-03 | Conformité RGPD (registre, droits, anonymisation) | `UserModule`, `NotificationModule`, `PaymentModule` | § 6.4 (champs marqués « Sensibilité RGPD : Oui »), § 9.3 | —                                    |
| ENF-04 | Sécurité (STRIDE flux critiques, défense en profondeur) | `AuthModule`, `TicketModule`, `PaymentModule` | § 9.1 (STRIDE flux 1 & 2), § 7.1, § 8 (HMAC webhook, JWT) | ADR-003                              |
| ENF-05 | Observabilité (logs, métriques, traces W3C) | Tous les modules                          | § 8 (`traceId` Problem Details, en-têtes événements), § 9.1 (audit log) | —                                    |
| ENF-06 | Accessibilité (WCAG 2.1 AA)               | Front (hors périmètre backend DCT)          | ⚠ **Non couvert** dans cette DCT (front à traiter en TP 1.7 / 1.8) | —                                    |
| ENF-07 | Maintenabilité (CI/CD, tests, documentation) | Tous les modules                         | DCT entière + § 11 (qualité documentaire à produire en TP 1.7) | —                                    |
| ENF-08 | Internationalisation (FR + EN minimum)    | `NotificationModule`, Front                 | ⚠ **Non couvert** à ce stade — `template_code` § 6.4 supporte le multi-langue mais stratégie complète à formaliser en TP 1.8 | —                                    |

### §10.2 — Synthèse

#### Décompte de couverture

- **Exigences fonctionnelles (EF)** : **13 / 13** référencées dans la matrice. **2 dépendances modules à compléter** (`EventModule` et `PaymentModule` mentionnés mais fiches § 7 non encore produites — prévues en itération suivante § 7.4 et § 7.5).
- **Exigences non-fonctionnelles (ENF)** : **6 / 8 couvertes** (ENF-01 à ENF-05, ENF-07 partielle), **2 non couvertes** (ENF-06 accessibilité, ENF-08 i18n).

#### Exigences explicitement non couvertes à ce stade

| Réf.   | Intitulé                                  | Justification du report                                                                              | Échéance prévue                                  |
|--------|-------------------------------------------|------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| ENF-06 | Accessibilité WCAG 2.1 AA                | Périmètre front non documenté dans la DCT actuelle (focus backend en TP 1.4 → 1.6).                  | TP 1.7 / 1.8 — section dédiée § 9.4 à produire. |
| ENF-07 | Maintenabilité (qualité éditoriale + CI/CD) | Pipeline CI/CD documenté en partie via les conventions § 8 mais aucune section dédiée (lint, couverture cible, scan vulnérabilités). | TP 1.7 — section § 11 qualité documentaire et § 12 outillage. |
| ENF-08 | Internationalisation                      | Le `template_code` (§ 6.4 entité `Notification`) supporte structurellement le multi-langue, mais stratégie complète (sélection langue, gestion fallback, formats locale-dépendants) non formalisée. | TP 1.8 — section dédiée à produire.             |

#### Lecture par module (vérification d'absence d'orphelin)

| Module                    | Nombre d'exigences servies | Statut                                                                 |
|---------------------------|----------------------------|------------------------------------------------------------------------|
| `AuthModule`              | 2 (EF-01, ENF-04)          | OK — responsabilité unique respectée.                                   |
| `EventModule` *(stub)*    | 7 (EF-02 → EF-05, EF-09, EF-10, ENF-01) | ⚠ Apparaît sur 7 lignes — à surveiller en § 7.4 pour vérifier la responsabilité unique (peut justifier sous-modules `EventCatalogModule` + `EventLifecycleModule` si fourre-tout). |
| `TicketModule`            | 6 (EF-06, EF-07, EF-08, EF-13, EF-09, EF-10, ENF-01, ENF-04) | OK — cœur du domaine, cohérent avec la fiche § 7.2.                    |
| `PaymentModule` *(stub)*  | 3 (EF-07, EF-08, ENF-04)   | OK — fiche § 7 à produire en itération suivante.                        |
| `NotificationModule`      | 2 (EF-11, ENF-03 partiel)  | OK — cohérent avec la fiche § 7.3.                                      |
| `UserModule` *(stub)*     | 2 (EF-12, ENF-03)          | OK — fiche § 7 à produire en itération suivante.                        |

Aucun module orphelin (servant zéro exigence) n'est identifié. `EventModule` est le seul candidat potentiel au statut « fourre-tout » et fera l'objet d'un examen attentif lors de la rédaction de sa fiche § 7.4 — un découpage en deux sous-modules sera envisagé si la responsabilité unique n'est pas démontrable.

#### Cohérence transversale (vérification finale)

| Point de contrôle                                                                                              | État |
|----------------------------------------------------------------------------------------------------------------|------|
| Tous les modules cités existent ou sont explicitement marqués « à détailler en § 7 »                            | OK   |
| Tous les ADR cités (ADR-001, ADR-002, ADR-003) existent dans `/docs/adr/`                                       | OK   |
| Toutes les sections DCT citées (§ 6.4, § 7, § 8, § 9.1, § 9.2, § 9.3) existent dans le dépôt                    | OK   |
| Aucune EF marquée comme couverte ne pointe vers un module ou une section inexistante                            | OK   |
| Les ENF non couvertes (ENF-06, ENF-08) sont explicitement signalées avec échéance plutôt que masquées            | OK   |

---

*Dernière mise à jour : 2026-04-30*
