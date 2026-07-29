---
name: Ingest and maintain a restaurant menu on Just Eat
description: Publish a restaurant's menu to the Just Eat UK API and keep its order and service times current, for POS/partner integrations.
api: openapi/just-eat-uk-openapi-original.json
operations: [putMenuForIngestion, GetOrderTimes, UpdateOrderTime, putRestaurantServiceTimes]
---

# Ingest and maintain a restaurant menu (Just Eat UK)

Use this skill from a POS / partner integration to publish and maintain a
restaurant's menu and availability windows on the Just Eat UK API
(`https://uk.api.just-eat.io`).

## Auth
Partner write calls use the partner API key: `Authorization: JE-API-KEY <key>`.
Some sign-up flows use the restaurant JWT. See
`authentication/just-eat-authentication.yml`.

## Steps
1. **Publish the menu** — call `putMenuForIngestion`
   (`PUT /restaurants/{tenant}/{restaurantId}/menu`) with the full menu payload.
   Menu ingestion is asynchronous: watch for the `menu-ingestion-complete`
   webhook (see `asyncapi/just-eat-webhooks.yml`) to confirm processing.
2. **Set service times** — call `putRestaurantServiceTimes`
   (`PUT /restaurants/{tenant}/{restaurantId}/servicetimes`) to define when each
   service type is available.
3. **Read current lead times** — call `GetOrderTimes`
   (`GET /restaurants/{tenant}/{restaurantId}/ordertimes`) to see current
   delivery/collection lead times.
4. **Adjust a lead time** — call `UpdateOrderTime`
   (`PUT /restaurants/{tenant}/{restaurantId}/ordertimes/{dayOfWeek}/{serviceType}`)
   to change the lead time for a specific day and service type.

## Rules
- Always include the `{tenant}` path segment (country marketplace, e.g. `uk`).
- Menu ingestion completes via webhook, not synchronously — do not assume the
  menu is live on a 2xx from the PUT alone.
- Handle `409 Conflict` (concurrent update) and `429` (rate limit) with retry.
  See `errors/just-eat-problem-types.yml`. No idempotency key is supported, so
  guard client-side against duplicate submissions.
