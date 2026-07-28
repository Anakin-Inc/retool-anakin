# Anakin + Retool

Retool has no third-party integration marketplace to submit to — confirmed:
`retool.com/integrations` is a marketing catalog of APIs people connect via
Retool's generic REST API resource, not a directory vendors can list on, and
there's no PR-based or form-based submission mechanism for a new API
integration (their "Custom Component Library" system is for private
per-workspace UI components, and their Partner Program is a
business/agency relationship, not a technical listing). So there's nothing
to build or submit — the entire "integration" is this doc, showing users how
to wire Anakin into Retool themselves.

## Add Anakin as a REST API resource

1. In your Retool org: **Resources** → **Create New** → **REST API**.
2. **Base URL**: `https://api.anakin.io/v1`
3. **Headers** → add `X-API-Key` = your Anakin API key (get one at
   [anakin.io/dashboard](https://anakin.io/dashboard)).
4. Save the resource as `Anakin`.

## Example query — scrape a URL

Create a new query against the `Anakin` resource:

- **Method**: `POST`
- **Path**: `/url-scraper`
- **Body**:
  ```json
  { "url": {{ textInput1.value }}, "generateJson": true }
  ```
- This returns `{ "jobId": "...", "status": "pending" }` — add a second
  query (`GET /url-scraper/{{ query1.data.jobId }}`) to poll it, or trigger
  the second query on an interval/button.

## Example query — AI search (synchronous, no polling)

- **Method**: `POST`
- **Path**: `/search`
- **Body**:
  ```json
  { "prompt": {{ textInput1.value }}, "limit": 5 }
  ```
- Returns results immediately — no second query needed.

Full endpoint reference: [anakin.io/docs/api-reference](https://anakin.io/docs/api-reference).
