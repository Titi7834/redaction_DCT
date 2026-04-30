# Changelog DCT SupEvents

Format : [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/), respect [Semantic Versioning](https://semver.org/lang/fr/) appliqué à un document (incrément mineur à chaque TP, majeur à la soutenance).

## [0.7.0] — 2026-04-30 (TP 1.7 — Audit qualité & CI)

### Ajouté
- `00-page-garde.md` — page de garde + sommaire navigable (correctif audit S1 + S2).
- `00-glossaire.md` — glossaire de 23 termes techniques (correctif audit S4).
- `CHANGELOG.md` — historique versionné (correctif audit S5).
- `docs/audit-qualite.md` — grille d'audit complète, 30 critères / 60 points.
- `docs/adr/template.md` — gabarit ADR (recréé après suppression).
- `.github/CODEOWNERS` — attribution par section.
- `.github/workflows/docs-quality.yml` — pipeline CI (markdownlint + link-check + mermaid-cli).
- `.markdownlint.json` — configuration lint.
- `docs/ci-validation.md` — preuve de fonctionnement CI.
- Pied de page « Dernière mise à jour » sur chaque section principale (correctif audit M1).

### Modifié
- `08-interfaces-api.md` — ajout de la fiche événement `ticket.cancelled` manquante (correctif audit CO3).

### Connu / dette assumée
- Sections § 6.1, § 6.2, § 6.3 (vue logique / processus / déploiement) absentes — TP 1.8.
- Fiches modules `EventModule`, `PaymentModule`, `UserModule` absentes — TP 1.8.
- ENF-06 accessibilité, ENF-08 i18n non couvertes — TP 1.8.

## [0.6.0] — 2026-04-30 (TP 1.6 — Exigences transverses & traçabilité)

### Ajouté
- `09-exigences-transverses.md` — STRIDE flux 1 (inscription payante) + flux 2 (webhook Stripe), 3 SLO de performance avec budget d'erreur, registre RGPD + 3 procédures.
- `10-tracabilite.md` — matrice 13 EF + 8 ENF × DCT, synthèse honnête des exigences non couvertes.

## [0.5.0] — 2026-04-29 (TP 1.5 — Conception détaillée modules & ADR)

### Ajouté
- `07-modules.md` — fiches détaillées `AuthModule`, `TicketModule`, `NotificationModule`.
- `docs/adr/ADR-001.md` — verrou pessimiste PostgreSQL pour l'allocation d'une place.
- `docs/adr/ADR-002.md` — idempotence par clé client (`Idempotency-Key`).
- `docs/adr/ADR-003.md` — JWT stateless courte durée + refresh token rotatif.
- `docs/adr/README.md` — index ADR.

## [0.4.0] — 2026-04-29 (TP 1.4 — Modèle de données & contrats d'API)

### Ajouté
- `06.4-vue-donnees.md` — diagramme ER (9 entités), dictionnaire complet, stratégie de stockage, RGPD.
- `08-interfaces-api.md` — tableau synoptique 22 endpoints REST, 2 événements asynchrones documentés (`ticket.confirmed`, `payment.failed`), conventions transverses.

---

*Dernière mise à jour : 2026-04-30*
