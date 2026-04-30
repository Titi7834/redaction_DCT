## §8 — Interfaces & contrats d'API

**Fait par Tom LEPRIEUR, Arthur L'AFFETER et Tiago DA COSTA**

### Tableau synoptique des endpoints REST

Toutes les routes sont préfixées par `/api/v1/`. La colonne « Auth » précise le mode d'authentification exigé : `Public` (aucune), `JWT` (jeton d'accès porteur signé — algorithme de signature à figer en § 9, hypothèse V1 RS256), `JWT + role:organizer`, `JWT + role:admin`, ou `HMAC` (signature `Stripe-Signature` vérifiée côté serveur). La colonne « Codes retour » liste les statuts HTTP attendus pour les cas nominaux et les erreurs métier les plus structurantes (au-delà des erreurs techniques 5xx, qui font l'objet d'un traitement transverse en § 8.2).

| Méthode | Chemin                                              | Description                                                       | Auth                       | Codes retour                              | Dépendances aval                                  |
|---------|-----------------------------------------------------|-------------------------------------------------------------------|----------------------------|-------------------------------------------|---------------------------------------------------|
| POST    | /api/v1/auth/oidc/callback                          | Callback du flux OpenID Connect, échange du code contre un JWT    | Public                     | 200, 400, 401, 502                        | IdentityProvider (OIDC), UserService              |
| POST    | /api/v1/auth/refresh                                | Échange un refresh token contre une nouvelle paire de jetons      | Public (cookie HttpOnly)   | 200, 401, 403                             | AuthService, Redis (rotation)                     |
| POST    | /api/v1/auth/logout                                 | Révoque le refresh token courant                                  | JWT                        | 204, 401                                  | AuthService, Redis                                |
| GET     | /api/v1/events                                      | Liste paginée des événements publiés                              | Public                     | 200, 400                                  | EventService, Redis (cache)                       |
| GET     | /api/v1/events/search                               | Recherche plein texte + filtres (catégorie, date, lieu)           | Public                     | 200, 400                                  | EventService, Redis (cache)                       |
| GET     | /api/v1/events/{eventId}                            | Détail d'un événement publié                                      | Public                     | 200, 404                                  | EventService                                      |
| POST    | /api/v1/events                                      | Création d'un événement (statut `draft`)                          | JWT + role:organizer       | 201, 400, 401, 403, 422                   | EventService                                      |
| PATCH   | /api/v1/events/{eventId}                            | Mise à jour d'un événement non publié                             | JWT + role:organizer       | 200, 400, 401, 403, 404, 409, 422         | EventService                                      |
| POST    | /api/v1/events/{eventId}/publish                    | Publication de l'événement                                        | JWT + role:organizer       | 200, 401, 403, 404, 409, 422              | EventService                                      |
| POST    | /api/v1/events/{eventId}/cancel                     | Annulation d'un événement déjà publié                             | JWT + role:organizer       | 202, 401, 403, 404, 409                   | EventService, RabbitMQ (`event.cancelled`)        |
| POST    | /api/v1/events/{eventId}/tickets                    | Création d'une réservation (billet `reserved`)                    | JWT                        | 201, 400, 401, 403, 404, 409, 422         | TicketService, EventService, RabbitMQ             |
| GET     | /api/v1/tickets                                     | Liste des billets de l'utilisateur courant                        | JWT                        | 200, 401                                  | TicketService                                     |
| GET     | /api/v1/tickets/{ticketId}                          | Détail d'un billet (incluant statut)                              | JWT                        | 200, 401, 403, 404                        | TicketService                                     |
| DELETE  | /api/v1/tickets/{ticketId}                          | Annulation d'un billet par son détenteur                          | JWT                        | 204, 401, 403, 404, 409                   | TicketService, PaymentService                     |
| POST    | /api/v1/tickets/{ticketId}/payment-intent           | Initiation d'un paiement Stripe pour un billet `reserved`         | JWT                        | 200, 401, 403, 404, 409, 422              | PaymentService, Stripe                            |
| POST    | /api/v1/payments/webhook                            | Réception des événements Stripe (PaymentIntent.*)                 | HMAC (Stripe-Signature)    | 200, 400, 401                             | PaymentService, RabbitMQ (`ticket.confirmed`, `payment.failed`) |
| GET     | /api/v1/organizer/events                            | Tableau de bord — événements organisés par l'utilisateur courant  | JWT + role:organizer       | 200, 401, 403                             | EventService, AnalyticsService                    |
| GET     | /api/v1/organizer/events/{eventId}/participants     | Liste paginée des participants d'un événement                     | JWT + role:organizer       | 200, 401, 403, 404                        | TicketService                                     |
| GET     | /api/v1/organizer/events/{eventId}/participants.csv | Export CSV des participants                                       | JWT + role:organizer       | 200, 401, 403, 404                        | TicketService, S3                                 |
| GET     | /api/v1/organizer/events/{eventId}/kpis             | Indicateurs de performance (taux de remplissage, CA, no-show)     | JWT + role:organizer       | 200, 401, 403, 404                        | AnalyticsService                                  |
| POST    | /api/v1/admin/organizers/{organizerId}/validate     | Validation administrative d'un compte organisateur                | JWT + role:admin           | 200, 401, 403, 404, 409                   | OrganizerService, NotificationService             |
| POST    | /api/v1/admin/organizers/{organizerId}/revoke       | Révocation d'un compte organisateur                               | JWT + role:admin           | 200, 401, 403, 404, 409                   | OrganizerService, NotificationService             |

### Conventions transverses

#### Format des erreurs (RFC 7807 Problem Details)

Toutes les réponses d'erreur respectent `application/problem+json` :

```json
{
  "type": "https://api.supevents.fr/errors/ticket-conflict",
  "title": "Ticket already exists for this event",
  "status": 409,
  "detail": "User 9f1c… already holds an active ticket for event b73a….",
  "instance": "/api/v1/events/b73a…/tickets",
  "code": "TICKET_DUPLICATE",
  "traceId": "0e2f8b…"
}
```

Le champ `code` est stable et documenté dans le catalogue d'erreurs (§ 8.4). Le `traceId` correspond à l'identifiant W3C `traceparent` propagé par la couche d'observabilité (outillage à arbitrer en § 10, hypothèse V1 OpenTelemetry), ce qui permet le rapprochement avec les journaux applicatifs.

#### Stratégie de versioning

- Versioning par segment d'URL : `/api/v1/`, `/api/v2/`. La durée minimale de cohabitation des versions reste à arbitrer (hypothèse V1 : au moins 6 mois après l'annonce de dépréciation, à valider avec le métier).
- En-tête `Sunset` (RFC 8594) renvoyé sur les routes dépréciées avec la date d'arrêt.
- Les changements rétro-incompatibles déclenchent une nouvelle version. L'ajout de champs facultatifs reste rétro-compatible.

#### Rate limiting et quotas

- Quotas par défaut : ordres de grandeur de quelques dizaines de requêtes/minute/IP pour les routes publiques et de quelques centaines de requêtes/minute/utilisateur authentifié (valeurs précises à régler en exploitation, à confronter au CDC § perf).
- En-têtes de réponse normalisés : `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`.
- Dépassement : code 429 + `Problem Details` (`code: "RATE_LIMITED"`) + en-tête `Retry-After`.
- Les routes coûteuses (`/events/search`, `/organizer/.../participants.csv`) ont des quotas dédiés stockés dans Redis (compteurs glissants), à calibrer après mesure.

#### Justification des choix d'API structurants

Trois décisions de découpage portent une intention métier qu'OpenAPI ne capture pas et qui méritent d'être explicitées.

- **`POST /events/{eventId}/cancel` plutôt que `PATCH /events/{eventId}` avec `{ status: "cancelled" }`.** L'annulation est une opération métier qui déclenche une cascade : publication d'`event.cancelled`, remboursements automatiques des billets payés, notifications utilisateurs. L'exposer comme une action dédiée rend le contrat lisible (le client sait qu'il s'agit d'une opération asynchrone et coûteuse, code 202), simplifie l'autorisation (politique RBAC dédiée), et empêche le client de tenter une mise à jour partielle qui contournerait la machine à états.
- **`POST /tickets/{ticketId}/payment-intent` séparé de `POST /events/{eventId}/tickets`.** La réservation et le paiement sont deux opérations distinctes : une réservation peut exister sans paiement (événement gratuit, abandon en cours de tunnel), un paiement n'a de sens qu'en référence à un billet existant. Les fusionner forcerait le client à gérer deux états d'erreur sur un seul appel et empêcherait la reprise de paiement après abandon.
- **`POST /payments/webhook` exposé en racine et non sous `/payments/{id}/...`.** Stripe pousse les événements sans connaître nos identifiants internes — la résolution se fait sur `stripe_payment_intent_id` côté serveur. Imposer un `id` interne dans l'URL forcerait une indirection inutile et créerait un couplage fragile.

#### Contrats inter-services

Les services internes communiquent via HTTP/gRPC interne ou via RabbitMQ. Le tableau ci-dessous formalise les **objectifs cibles** retenus comme hypothèses de dimensionnement V1 — les valeurs précises (latences, timeouts, seuils de circuit breaker) seront ajustées après mesure en pré-production et validées par le CDC § perf et § exploitation.

| Appelant                  | Appelé              | Type     | Objectif latence p95           | Timeout client (hypothèse) | Politique de retry (hypothèse)         | Coupe-circuit (hypothèse)                              |
|---------------------------|---------------------|----------|--------------------------------|----------------------------|----------------------------------------|--------------------------------------------------------|
| `TicketService`           | `EventService`      | gRPC     | sub-100 ms                     | quelques centaines de ms   | Quelques retries courts, opération idempotente uniquement | Ouverture sur taux d'échec élevé sur fenêtre courte    |
| `TicketService`           | `PaymentService`    | gRPC     | sub-200 ms                     | sub-seconde                | Retry unique idempotent                | Ouverture sur taux d'échec élevé sur fenêtre courte    |
| `PaymentService`          | Stripe API          | HTTPS    | sub-500 ms                     | quelques secondes          | Retries exponentiels (cf. SDK Stripe)  | Bulkhead nombre de connexions ; circuit ouvert sur 5xx persistants |
| `NotificationService`     | Provider e-mail     | HTTPS    | sub-seconde                    | quelques secondes          | Retries exponentiels avec jitter       | Bascule provider secondaire après seuil d'échec        |
| `OrganizerDashboardService` | `AnalyticsService` | gRPC     | sub-300 ms                     | sub-seconde                | Retry unique                           | Dégradation gracieuse : KPI servis depuis cache Redis  |
| Tous les producteurs      | RabbitMQ            | AMQP     | publish ack quasi-immédiat     | sub-seconde                | Outbox pattern + republish job périodique | Outbox PostgreSQL en cas d'indisponibilité broker     |

Les valeurs cibles listées sont des hypothèses internes à valider — toute consommation au-delà déclenchera une alerte exploitation et une revue d'architecture conjointe entre les équipes appelante et appelée.

### Événements asynchrones

Tous les événements métier sont publiés sur RabbitMQ via des exchanges `topic` versionnés. Chaque message porte les en-têtes standards `x-event-id` (UUID), `x-event-name`, `x-event-version`, `x-trace-id`, `x-correlation-id`. Le nom de l'événement est au passé : il décrit un fait observé, pas une intention. Les payloads ne contiennent que des identifiants et des codes de statut — aucun champ marqué « Sensibilité RGPD : Oui » en § 6.4 n'y figure.

#### `ticket.confirmed`

##### Fiche descriptive

| Champ                  | Valeur                                                                                                                                                                                                                                                                                                |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Nom                    | `ticket.confirmed`                                                                                                                                                                                                                                                                                    |
| Producteur             | `TicketService` — publié lorsqu'un billet bascule en statut `confirmed`, soit immédiatement pour les événements gratuits, soit à la réception d'un `payment_intent.succeeded` côté webhook Stripe relayé par `PaymentService`.                                                                       |
| Topic / exchange       | Exchange `tickets.events.v1` (type `topic`) — routing key `ticket.confirmed`.                                                                                                                                                                                                                          |
| Consommateurs connus   | `NotificationService` (envoie l'e-mail de confirmation + le PDF du billet), `AnalyticsService` (incrément des compteurs taux de remplissage / CA), `OrganizerDashboardService` (rafraîchit la liste des participants en temps réel), `WaitingListService` (no-op ; ignore mais conservé pour audit). |
| Garantie de livraison  | At-least-once. Les consommateurs sont idempotents via la clé `x-event-id` (déduplication Redis ; durée de rétention de la clé à aligner avec la fenêtre maximale de retry, hypothèse V1 quelques jours).                                                                                              |
| Stratégie de retry     | Backoff exponentiel avec jitter sur plusieurs paliers (de l'ordre de la seconde à l'heure), nombre maximal de tentatives à figer en exploitation (hypothèse V1 : ≈ 6). Au-delà, message routé vers la DLQ `tickets.events.v1.dlq` ; alerte exploitation si la DLQ se remplit anormalement (seuil à calibrer). |

##### Schéma JSON du payload

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TicketConfirmedEvent",
  "type": "object",
  "required": ["eventId", "version", "occurredAt", "ticketId", "userId", "supEventId", "channel", "amountCents", "currency"],
  "properties": {
    "eventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique du message (clé de déduplication)."
    },
    "version": {
      "type": "string",
      "const": "1.0",
      "description": "Version du contrat d'événement."
    },
    "occurredAt": {
      "type": "string",
      "format": "date-time",
      "description": "Horodatage ISO 8601 du fait observé."
    },
    "ticketId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du billet confirmé."
    },
    "userId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du détenteur du billet."
    },
    "supEventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'événement métier (Event)."
    },
    "channel": {
      "type": "string",
      "enum": ["free", "paid"],
      "description": "Voie de confirmation : gratuit ou payé."
    },
    "amountCents": {
      "type": "integer",
      "minimum": 0,
      "description": "Montant facturé en centimes (0 si gratuit)."
    },
    "currency": {
      "type": "string",
      "pattern": "^[A-Z]{3}$",
      "description": "Devise ISO 4217."
    },
    "paymentId": {
      "type": ["string", "null"],
      "format": "uuid",
      "description": "Identifiant du paiement associé (null si gratuit)."
    },
    "correlationId": {
      "type": "string",
      "format": "uuid",
      "description": "Corrélation de bout en bout (propagation W3C trace)."
    }
  },
  "additionalProperties": false
}
```

#### `payment.failed`

##### Fiche descriptive

| Champ                  | Valeur                                                                                                                                                                                                                                                                                          |
|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Nom                    | `payment.failed`                                                                                                                                                                                                                                                                                |
| Producteur             | `PaymentService` — publié à la réception d'un webhook Stripe `payment_intent.payment_failed`, ou à l'expiration d'un time-out interne sur le PaymentIntent (durée précise à figer en § 8.4, hypothèse V1 de l'ordre du quart d'heure).                                                          |
| Topic / exchange       | Exchange `payments.events.v1` (type `topic`) — routing key `payment.failed`.                                                                                                                                                                                                                    |
| Consommateurs connus   | `TicketService` (rebascule le billet `reserved` vers `cancelled` et libère la place), `NotificationService` (envoie un e-mail d'échec + lien de relance), `WaitingListService` (déclenche la promotion de la première entrée éligible), `AnalyticsService` (incrément des KPI taux d'échec). |
| Garantie de livraison  | At-least-once. Idempotence par `x-event-id` ; les consommateurs vérifient le statut courant du billet avant action pour gérer les arrivées tardives.                                                                                                                                            |
| Stratégie de retry     | Backoff exponentiel avec jitter, nombre maximal de tentatives à figer en exploitation (hypothèse V1 : ≈ 5). DLQ `payments.events.v1.dlq` ; alerte critique sur remplissage anormal de la DLQ — un blocage prolongé empêcherait la libération des places (priorité d'astreinte à arbitrer).      |

##### Schéma JSON du payload

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "PaymentFailedEvent",
  "type": "object",
  "required": ["eventId", "version", "occurredAt", "paymentId", "ticketId", "userId", "supEventId", "amountCents", "currency", "failureReason"],
  "properties": {
    "eventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique du message (clé de déduplication)."
    },
    "version": {
      "type": "string",
      "const": "1.0",
      "description": "Version du contrat d'événement."
    },
    "occurredAt": {
      "type": "string",
      "format": "date-time",
      "description": "Horodatage ISO 8601 du fait observé."
    },
    "paymentId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant interne du paiement."
    },
    "ticketId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du billet associé."
    },
    "userId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du détenteur."
    },
    "supEventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'événement métier (Event)."
    },
    "amountCents": {
      "type": "integer",
      "minimum": 0,
      "description": "Montant tenté en centimes."
    },
    "currency": {
      "type": "string",
      "pattern": "^[A-Z]{3}$",
      "description": "Devise ISO 4217."
    },
    "failureReason": {
      "type": "string",
      "enum": [
        "card_declined",
        "insufficient_funds",
        "expired_card",
        "authentication_required",
        "processing_error",
        "timeout",
        "other"
      ],
      "description": "Cause normalisée de l'échec (mappée depuis Stripe)."
    },
    "stripeErrorCode": {
      "type": ["string", "null"],
      "description": "Code d'erreur natif Stripe pour observabilité."
    },
    "retryable": {
      "type": "boolean",
      "description": "Indique si une nouvelle tentative côté utilisateur est pertinente."
    },
    "correlationId": {
      "type": "string",
      "format": "uuid",
      "description": "Corrélation de bout en bout (propagation W3C trace)."
    }
  },
  "additionalProperties": false
}
```

#### `ticket.cancelled`

##### Fiche descriptive

| Champ                  | Valeur                                                                                                                                                                                                                                                                                                                              |
|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Nom                    | `ticket.cancelled`                                                                                                                                                                                                                                                                                                                  |
| Producteur             | `TicketService` — publié dans trois scénarios : (1) annulation par l'utilisateur (`DELETE /tickets/{id}`), (2) consommation de `payment.failed` sur un billet `reserved` (libération automatique), (3) consommation de `event.cancelled` (annulation en masse).                                                                     |
| Topic / exchange       | Exchange `tickets.events.v1` (type `topic`) — routing key `ticket.cancelled`.                                                                                                                                                                                                                                                       |
| Consommateurs connus   | `NotificationService` (envoie un e-mail d'annulation au détenteur), `PaymentService` (déclenche le remboursement Stripe si `reason = user_cancelled` ou `event_cancelled` et que le paiement avait été capturé), `WaitingListService` (promeut la première entrée éligible de la file d'attente), `AnalyticsService` (KPI annulation). |
| Garantie de livraison  | At-least-once. Idempotence par `x-event-id` ; les consommateurs vérifient le statut courant avant action.                                                                                                                                                                                                                            |
| Stratégie de retry     | Backoff exponentiel avec jitter ; max tentatives ≈ 6 (hypothèse V1, cohérent avec `ticket.confirmed`). DLQ `tickets.events.v1.dlq` partagée ; alerte exploitation sur remplissage anormal.                                                                                                                                          |

##### Schéma JSON du payload

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TicketCancelledEvent",
  "type": "object",
  "required": ["eventId", "version", "occurredAt", "ticketId", "userId", "supEventId", "reason"],
  "properties": {
    "eventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique du message (clé de déduplication)."
    },
    "version": {
      "type": "string",
      "const": "1.0",
      "description": "Version du contrat d'événement."
    },
    "occurredAt": {
      "type": "string",
      "format": "date-time",
      "description": "Horodatage ISO 8601 du fait observé."
    },
    "ticketId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du billet annulé."
    },
    "userId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du détenteur."
    },
    "supEventId": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'événement métier (Event)."
    },
    "reason": {
      "type": "string",
      "enum": ["user_cancelled", "payment_failed", "event_cancelled", "admin_revoked"],
      "description": "Cause normalisée de l'annulation."
    },
    "previousStatus": {
      "type": "string",
      "enum": ["reserved", "confirmed"],
      "description": "Statut du billet juste avant l'annulation (utile pour décider d'un remboursement)."
    },
    "refundRequired": {
      "type": "boolean",
      "description": "Indique si un remboursement Stripe doit être déclenché par PaymentService."
    },
    "paymentId": {
      "type": ["string", "null"],
      "format": "uuid",
      "description": "Identifiant du paiement associé (null si billet gratuit)."
    },
    "correlationId": {
      "type": "string",
      "format": "uuid",
      "description": "Corrélation de bout en bout (propagation W3C trace)."
    }
  },
  "additionalProperties": false
}
```

### Cohérence § 6.4 ↔ § 8 (auto-revue)

| Point de contrôle                                                                                                              | État |
|--------------------------------------------------------------------------------------------------------------------------------|------|
| Toutes les entités référencées dans les payloads existent au dictionnaire (`User`, `Ticket`, `Event`, `Payment`)               | OK   |
| Tous les `id` côté API sont en UUID (cohérent avec les PK PostgreSQL UUID v4)                                                  | OK   |
| Chaque contrainte UNIQUE en base est associée à un 409 documenté côté API (ticket actif unique → 409 sur `POST /tickets`)      | OK   |
| Aucun champ marqué « Sensibilité RGPD : Oui » ne fuit dans les payloads (les événements ne portent que des `id` et des codes)  | OK   |
| Webhook Stripe authentifié par HMAC `Stripe-Signature`, jamais par JWT                                                         | OK   |
| Chaque événement a au moins un consommateur identifié — pas d'événement orphelin                                               | OK   |

> **Note sur les valeurs chiffrées de cette section.** Toutes les valeurs numériques précises (quotas, latences cibles, timeouts, fenêtres de retry, durées de rétention) sont présentées comme des **hypothèses V1**, à confronter au CDC SupEvents (§ perf, § exploitation, § sécurité) et à ajuster après mesure réelle en pré-production.

---

*Dernière mise à jour : 2026-04-30*
