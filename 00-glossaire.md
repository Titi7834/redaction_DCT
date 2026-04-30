# Glossaire SupEvents

Termes techniques et acronymes utilisés dans la DCT, classés par ordre alphabétique. Ce glossaire est mis à jour à chaque ajout de terme structurant.

| Terme / Acronyme    | Définition                                                                                                                                               | Apparaît dans               |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|
| **3DS2**            | 3-D Secure 2.0 — protocole d'authentification renforcée du paiement par carte, exigé par la directive européenne DSP2. Géré par Stripe côté client.     | § 6.4, § 9.1                |
| **ADR**             | Architecture Decision Record — document court (format Nygard) capturant une décision architecturale, son contexte, ses alternatives, ses conséquences. | `/docs/adr/`, § 7, § 10     |
| **at-least-once**   | Garantie de livraison messagerie : chaque message est livré au moins une fois (avec risque de duplication, à neutraliser par idempotence consommateur).  | § 8                         |
| **Backoff exponentiel** | Stratégie de retry où le délai entre deux tentatives croît de façon exponentielle (avec jitter recommandé pour éviter les rafales synchronisées).     | § 7.3, § 8                  |
| **Budget d'erreur** | Tolérance d'indisponibilité dérivée d'un SLO. SLO 99,5 % → 216 min/mois consommables par incidents et déploiements.                                       | § 9.2                       |
| **DLQ**             | Dead Letter Queue — file de débordement RabbitMQ recevant les messages dont le traitement a échoué après le nombre maximal de tentatives.                  | § 7.3, § 8                  |
| **HMAC**            | Hash-based Message Authentication Code — signature symétrique d'un message à l'aide d'un secret partagé (HMAC-SHA256 pour le webhook Stripe).             | § 8, § 9.1.2                |
| **Idempotence**     | Propriété d'une opération qui produit le même résultat qu'elle soit appelée une ou plusieurs fois avec les mêmes paramètres.                              | ADR-002, § 7, § 8           |
| **Idempotency-Key** | En-tête HTTP standard (Stripe, AWS, GitHub) porté par le client pour signaler qu'une requête est rejouable sans effet de bord.                              | ADR-002, § 8                |
| **JWT**             | JSON Web Token — jeton compact signé, autoportant, transporté typiquement en `Authorization: Bearer`. Utilisé en SupEvents pour l'access token applicatif.| ADR-003, § 7.1, § 8         |
| **OIDC**            | OpenID Connect — couche d'authentification au-dessus d'OAuth 2.0. SupEvents délègue l'authentification au SSO de l'école via OIDC.                         | § 7.1, § 8                  |
| **Outbox pattern**  | Pattern transactionnel : l'événement à publier est inséré dans une table « outbox » dans la même transaction que la mise à jour métier ; un worker le publie ensuite. Garantit qu'aucun événement n'est perdu en cas de crash. | § 9.1.2                     |
| **PCI-DSS**         | Payment Card Industry Data Security Standard — référentiel de sécurité pour le traitement des données de carte bancaire. SupEvents délègue la conformité à Stripe. | § 6.4, § 9.1                |
| **PII**             | Personally Identifiable Information — donnée personnelle identifiable au sens RGPD (nom, e-mail, IP, etc.).                                              | § 6.4, § 9.3                |
| **PKCE**            | Proof Key for Code Exchange — extension OAuth 2.0 qui neutralise le risque de fuite du code d'autorisation OIDC.                                          | § 7.1                       |
| **Problem Details** | Format normalisé RFC 7807 pour les réponses HTTP d'erreur (`application/problem+json`), avec champs `type`, `title`, `status`, `detail`, `instance`.    | § 8                         |
| **RGPD**            | Règlement Général sur la Protection des Données — règlement européen 2016/679 encadrant le traitement des données personnelles.                          | § 6.4, § 9.3                |
| **Refresh token**   | Jeton longue durée permettant d'obtenir un nouveau JWT sans réauthentification complète. Stocké hashé en base (cf. § 6.4) et rotaté à chaque usage.       | ADR-003, § 7.1              |
| **SLI / SLO / SLA** | Service Level Indicator (mesure) / Objective (cible interne) / Agreement (engagement contractuel). La DCT documente les SLO.                              | § 9.2                       |
| **SSO**             | Single Sign-On — authentification unique permettant à l'utilisateur de se connecter à plusieurs applications via un même fournisseur d'identité.          | § 7.1                       |
| **STRIDE**          | Modèle d'analyse de menaces Microsoft : Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege.              | § 9.1                       |
| **Verrou pessimiste** | Stratégie de gestion de la concurrence où la ressource est verrouillée explicitement (`SELECT ... FOR UPDATE`) avant lecture/modification.               | ADR-001, § 7.2              |
| **WCAG**            | Web Content Accessibility Guidelines — référentiel d'accessibilité du W3C. Niveau AA exigé en France pour les services publics.                           | § 9 (à produire en TP 1.8)  |

---

*Dernière mise à jour : 2026-04-30*
