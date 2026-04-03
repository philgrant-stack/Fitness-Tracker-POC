# Squad Fitness — API Specification

*Version: 1.0 | Last updated: 3 April 2026*

This document defines the contract between the Squad Fitness frontend (`fitness-tracker.html`) and the GCP Cloud Function backend. Both sides should be built and tested against this spec.

---

## Base URL

```
https://REGION-PROJECT_ID.cloudfunctions.net/fitness-api
```

> Fill in `REGION` and `PROJECT_ID` after deploying in Phase 3. Store the full URL as a constant at the top of `fitness-tracker.html`.

---

## Endpoints

### 1. POST /log

Writes a new fitness entry to Firestore.

**Request**

```
POST {BASE_URL}/log
Content-Type: application/json
```

**Request Body**

```json
{
  "user":     "Phil",
  "type":     "cycling",
  "duration": 45,
  "date":     "2026-04-03",
  "notes":    "Morning ride along the bay",
  "muscle":   null
}
```

| Field | Type | Required | Validation |
|---|---|---|---|
| `user` | string | Yes | Non-empty string |
| `type` | string | Yes | One of: `cycling`, `cardio`, `strength` |
| `duration` | number | Yes | One of: `30`, `45`, `60` |
| `date` | string | Yes | Format: `YYYY-MM-DD` |
| `notes` | string | No | May be empty string. Defaults to `""` if omitted |
| `muscle` | string or null | Conditional | Required when `type` is `strength`. One of: `Chest`, `Back`, `Legs`, `Shoulders`, `Arms`, `Core`. Must be `null` for other activity types |

**Success Response** `200 OK`

```json
{
  "status": "ok",
  "id": "aB3xK9mNpQ"
}
```

> `id` is the Firestore auto-generated document ID. Not used by the frontend currently but useful for future delete/edit operations.

**Error Responses**

| Status | Reason | Body |
|---|---|---|
| `400` | Missing or invalid field | `{ "error": "Missing required field: type" }` |
| `400` | Invalid `muscle` for strength | `{ "error": "muscle is required when type is strength" }` |
| `405` | Wrong HTTP method | `{ "error": "Method not allowed" }` |
| `500` | Firestore write failed | `{ "error": "Internal server error" }` |

---

### 2. GET /entries

Returns all fitness entries from Firestore, sorted newest first.

**Request**

```
GET {BASE_URL}/entries
```

No request body or query parameters required.

**Success Response** `200 OK`

```json
[
  {
    "id":       "aB3xK9mNpQ",
    "user":     "Phil",
    "type":     "cycling",
    "duration": 45,
    "date":     "2026-04-03",
    "notes":    "Morning ride along the bay",
    "muscle":   null
  },
  {
    "id":       "cD7yL2nRsT",
    "user":     "Alex",
    "type":     "strength",
    "duration": 60,
    "date":     "2026-04-02",
    "notes":    "Feeling strong",
    "muscle":   "Chest"
  }
]
```

> Sorted by `date` descending (newest first). Where multiple entries share the same date, secondary sort is by `created_at` descending (server-set timestamp).

> Returns an empty array `[]` if no entries exist — never returns `null`.

**Error Responses**

| Status | Reason | Body |
|---|---|---|
| `405` | Wrong HTTP method | `{ "error": "Method not allowed" }` |
| `500` | Firestore read failed | `{ "error": "Internal server error" }` |

---

## Firestore Data Model

**Collection:** `fitness_entries`

Each document is stored with an auto-generated ID and the following fields:

```
fitness_entries/
  {auto-id}/
    user:       string        — e.g. "Phil"
    type:       string        — "cycling" | "cardio" | "strength"
    duration:   number        — 30 | 45 | 60
    date:       string        — "YYYY-MM-DD"
    notes:      string        — "" if not provided
    muscle:     string|null   — muscle group or null
    created_at: timestamp     — set server-side by Cloud Function on write
```

> `created_at` is set by the Cloud Function using `firestore.SERVER_TIMESTAMP` — it is never sent by the frontend. It provides a reliable sort key independent of the user-supplied `date` field.

---

## CORS

The Cloud Function must include CORS headers to allow the frontend to call it from a browser. The frontend is served from a different origin (GitHub Pages or similar), so without these headers all requests will be blocked.

Required response headers on every response:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type
```

The function must also handle `OPTIONS` preflight requests and return `200` with the above headers.

---

## Frontend Integration Points

Two locations in `fitness-tracker.html` need to be updated when the Cloud Function is deployed:

**1. Add the base URL constant** near the top of the `<script>` block:

```javascript
const API_URL = 'https://REGION-PROJECT_ID.cloudfunctions.net/fitness-api';
```

**2. Replace the TODO in `logEntry()`** (currently line ~167):

```javascript
// TODO: POST to Cloud Function here
// fetch(CLOUD_FUNCTION_URL + '/log', { method: 'POST', body: JSON.stringify(entry) })
```

Replace with:

```javascript
await fetch(`${API_URL}/log`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(entry)
});
```

**3. Load entries on app start** — add a `loadEntries()` function that calls `GET /entries` and replaces the `demoData` array:

```javascript
async function loadEntries() {
  const res = await fetch(`${API_URL}/entries`);
  entries = await res.json();
  renderFeed();
}
```

---

## Not In Scope (v1)

The following are intentionally excluded from this version to keep the build simple:

- Authentication — any user can log as any name
- Pagination — `/entries` returns all records every time
- Filtering server-side — all filtering (by user, by week) is done client-side
- Delete or edit endpoints
- A `/stats` endpoint — stats are computed client-side from the full dataset
