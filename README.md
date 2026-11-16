# awarddelta

What actually changed in federal contracting since you last looked — not a fresh dump of everything, just the delta? awarddelta is a local forwarding client for the hosted GovCon delta-feed MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

The core read is `whats_changed_since`: hand it the cursor from a previous response and it returns only the awards that are new or modified since that bookmark, so a business-development team can poll for movement instead of re-scanning the whole awards landscape. Around it sit a weekly digest that also flags contracts expiring soon (recompete opportunities), a watch that follows one named contractor's recent public award history, and a direct live search of the public USAspending awards dataset. Everything here reads published federal award data; the cursor is yours to keep — the client stores nothing between calls.

All 4 captured tools are built; there are no omissions.

## The tools

- **`whats_changed_since`** — Federal contract awards that are new or modified since a cursor token. Pass the `cursor` from a prior call to get only the changes since then (omit it for a first call). Filter by `naics` code prefixes, awarding `agency`, `recipient` contractor name, `keyword`, and a `minAmount`/`maxAmount` dollar range; cap rows with `limit` (default 100). No required parameters.
- **`delta_digest`** — A weekly delta digest for a filter: new awards, modifications, and contracts expiring in the next N days. The same NAICS / agency / recipient / keyword / dollar filters apply, plus `days` for the lookback window (default 7) and `expiringDays` for the expiry horizon (90 or 180, default 90). No required parameters.
- **`competitor_watch`** — Given a `contractor` name (the one required parameter), that company's recent federal awards over the last 180 days, plus any win, lose, or modify events the feed has accrued for them over time.
- **`search_awards`** — A direct live search of US federal awards in USAspending.gov, filterable by `naics` codes, `agency`, `recipient`, `keyword`, a `start`/`end` date range, and a `minAmount`/`maxAmount` dollar range, with a `limit` on rows returned (default 50, max 200). No required parameters.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/awarddelta mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "awarddelta": {
      "command": "npx",
      "args": ["-y", "@reneturcios/awarddelta", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/awarddelta
docker run --rm ghcr.io/cloudcorprecords/awarddelta delta_digest '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
awarddelta whats_changed_since '{"cursor":"eyJvZmZzZXQiOjEwMH0","naics":["5415"]}'
awarddelta delta_digest '{"agency":"Defense","expiringDays":180}'
awarddelta competitor_watch '{"contractor":"ECS Federal"}'
awarddelta search_awards '{"keyword":"cloud migration","minAmount":1000000}'
awarddelta --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `GOVCONFEED_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://govcon.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `awarddelta error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (all 4 tools, deep-equal, in order, with no titles invented and no `required` key added to the three tools that carry none), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
