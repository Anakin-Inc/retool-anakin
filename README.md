# Anakin + Retool

Use [Anakin](https://anakin.io) (web scraping, AI search, crawling, site mapping
and Wire actions, returned as markdown or JSON) inside Retool apps, workflows
and agents.

Retool has no third-party integration marketplace. `retool.com/integrations` is
Retool's own catalog with no vendor submission route (checked 2026-10-16), so Anakin connects
through Retool's built-in resource types. Pick the one that fits:

| Retool resource | Best for | Auth |
|---|---|---|
| [MCP Server](#option-1-mcp-server-resource-agents-and-app-functions) | Retool Agents, app functions, the app-building agent | OAuth (sign in to Anakin, no key to paste) |
| [REST API](#option-2-rest-api-resource) | Apps and workflows, with one shared API key | API key |

You need an Anakin account. The free tier includes 300 credits and needs no
card: sign up at [anakin.io/dashboard](https://anakin.io/dashboard), where you
also create an API key for option 2.

## Option 1: MCP Server resource (agents and app functions)

Anakin hosts a remote MCP server at `https://mcp.anakin.io/mcp` (Streamable
HTTP, OAuth 2.1 with dynamic client registration). It exposes the same tools as
the `@anakin-io/mcp` package: `scrape`, `search`, `map`, `crawl`,
`agentic_search`, `browser_task`, the `wire_*`, `monitor_*`, `session_*` and
`ai_visibility_*` tools.

1. **Resources** → **Create new** → **Resource** → **MCP Server**.
2. **Name**: `Anakin`.
3. **Server URL**: `https://mcp.anakin.io/mcp`
4. **Authentication**: **Automatic**, then click **Auto-configure
   authentication**. Retool reads the server's OAuth metadata and registers
   itself with Anakin.
5. **Create resource**, then **Connect with OAuth**, sign in to Anakin and
   approve access.
6. **Test connection** should list the Anakin tools.

Then:

- **Retool Agents**: in the agent's **Tools**, choose **Connect to MCP
  Server** and select the `Anakin` resource. The agent can now scrape pages,
  search the web and run crawls on its own.
- **Apps**: call a tool from a function on the `Anakin` resource. Each tool is
  a method under `tools` (the editor autocompletes the names), and the result
  is in `data`.

Notes:

- OAuth resources are per user: each Retool user who runs an Anakin tool
  connects their own Anakin account once. Use option 2 if an app should run
  on one shared key.
- Retool's tool timeout is at most 2 minutes. For long jobs (big crawls,
  `agentic_search`, `browser_task`), let the agent start the job and check on
  it, rather than waiting in one call.

## Option 2: REST API resource

1. **Resources** → **Create new** → **Resource** → **REST API**.
2. **Name**: `Anakin`.
3. **Query mode**: either
   - **Use an API spec** with **Spec URL** `https://anakin.io/openapi.json`,
     then **Load spec**. Retool reads every Anakin endpoint from the spec and
     sets the base URL (`https://api.anakin.io/v1`) for you. Recommended.
   - or enter the **Base URL** `https://api.anakin.io/v1` by hand.
4. **Authentication**: **None**. Under **URL parameters and headers**, add the
   header `X-API-Key` with your Anakin API key (`ask_...`). Keep the key in a
   Retool secret or configuration variable rather than in plain text.
5. **Test connection**, then **Create resource**.

### Scrape a page (one query, no polling)

- **Method**: `POST`
- **URL path**: `/url-scraper/scrape`
- **Body** (JSON):
  ```json
  { "url": {{ textInput1.value }}, "generateJson": false }
  ```
- Show the page with a Text component set to `{{ scrapeQuery.data.markdown }}`.
  With `"generateJson": true` the extracted fields are in
  `scrapeQuery.data.generatedJson`.
- Check `scrapeQuery.data.status`: a failed scrape also returns HTTP 200 with
  `status: "failed"` and an `error` message. If a page takes longer than about
  90 seconds, the response is HTTP 202 with an `id` and a non-final status;
  fetch the result later with `GET /url-scraper/{{ scrapeQuery.data.id }}`.
- Set `"useBrowser": true` for JavaScript-heavy pages (slower, more credits).
- This endpoint allows 20 requests per minute. For bulk jobs use
  `POST /url-scraper` (returns `{ "jobId": ... }`) and poll
  `GET /url-scraper/{jobId}`, for example from a Retool Workflow loop.

### AI web search

- **Method**: `POST`
- **URL path**: `/search`
- **Body** (JSON):
  ```json
  { "prompt": {{ textInput1.value }}, "limit": 5 }
  ```
- Returns immediately. Bind a Table to `{{ searchQuery.data.results }}`
  (each row has `url`, `title`, `snippet`, `date`). Costs 3 credits per call.

### Long-running jobs

`POST /crawl` and `POST /map` return `jobId`, `POST /agentic-search` returns
`job_id`. Run the start query, then a second `GET` query on the matching path
(for example `/crawl/{{ startCrawl.data.jobId }}` or
`/agentic-search/{{ startResearch.data.job_id }}`). Rerun the `GET` query
on an interval (the query's advanced settings) or from a button until
`status` is final, or poll from a Retool Workflow.

## Reference

- API reference: [anakin.io/docs/api-reference](https://anakin.io/docs/api-reference)
- OpenAPI spec: [anakin.io/openapi.json](https://anakin.io/openapi.json)
- Retool docs: [Connect to an external MCP server](https://docs.retool.com/data-sources/guides/connect/mcp),
  [Connect with an OpenAPI spec](https://docs.retool.com/data-sources/guides/connect/openapi),
  [Connect an MCP server to an agent](https://docs.retool.com/agents/guides/tools/connect-to-mcp-server)
