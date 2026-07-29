---
name: Discover restaurants and read their menu on Just Eat
description: Find restaurants serving a customer location on the Just Eat UK API and read a restaurant's menu, catalogue and service/order times.
api: openapi/just-eat-uk-openapi-original.json
operations: [SearchByPostcode, SearchByLocation, GetOrderTimes, getRestaurantServiceTimes]
---

# Discover restaurants and read their menu (Just Eat UK)

Use this skill to find restaurants that can serve a customer and inspect what
they offer. These are read operations against the Just Eat UK API
(`https://uk.api.just-eat.io`). All calls require HTTPS.

## Auth
Consumer/read calls use a Bearer JWT: `Authorization: Bearer <jwt>`.
Partner calls use the partner API key: `Authorization: JE-API-KEY <key>`.
See `authentication/just-eat-authentication.yml`.

## Steps
1. **Find restaurants by postcode** — call `SearchByPostcode`
   (`GET /restaurants/bypostcode/{postcode}`) with the customer postcode. Use
   this when you have a UK postcode.
2. **Or find by coordinates** — call `SearchByLocation`
   (`GET /restaurants/bylatlong`) with latitude/longitude when you only have a
   geolocation.
3. **Read lead times** — for a chosen restaurant call `GetOrderTimes`
   (`GET /restaurants/{tenant}/{restaurantId}/ordertimes`) to get delivery and
   collection lead times. `{tenant}` is the country marketplace (e.g. `uk`).
4. **Read service times** — call `getRestaurantServiceTimes`
   (`GET /restaurants/{tenant}/{restaurantId}/servicetimes`) to confirm when the
   restaurant is open for each service type.

## Rules
- Pass the `{tenant}` path segment on all restaurant-scoped calls.
- Handle `429 Too Many Requests` with backoff; `404` means the resource (or
  postcode coverage) was not found. See `errors/just-eat-problem-types.yml`.
- Dates/times are ISO 8601 with a UTC offset — convert to local for display.
