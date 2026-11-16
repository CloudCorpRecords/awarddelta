# BUILD REPORT — awarddelta

Clean-room rebuild of `com.dropwatchhq/govcon-feed`
v1.0.0 (remote-only MCP server at `https://govcon.dropwatchhq.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/awarddelta` 0.1.0, registry name
`io.github.CloudCorpRecords/awarddelta`.

## Surface built

Full surface: **all 4 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `whats_changed_since` | NO `required` key in the capture — served verbatim, none invented |
| `delta_digest` | NO `required` key in the capture — served verbatim, none invented |
| `competitor_watch` | `contractor` |
| `search_awards` | NO `required` key in the capture — served verbatim, none invented |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
keys, so none are served. The tool literals were generated
programmatically from
`harness/fixtures/govconfeed-tools-original.json` by the script at
`/tmp/b47-awarddelta-scratch/gen_forward.py` (parse + re-serialise
into `src/forward.ts`) — never retyped — and a post-generation check
(the generator's own round-trip assertion, the registration-integrity
test suite, plus the parity harness) confirmed the served literals
deep-equal the fixture's 4 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers), not
McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 4 tools' names,
descriptions, required fields (including the three tools with no
`required` key: `whats_changed_since`, `delta_digest`,
`search_awards`) and property shapes. The spec's note that the
capture carries no `title` keys matches the fixture exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://govcon.dropwatchhq.com/mcp`.
- Env override `GOVCONFEED_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`GOVCONFEED_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and
  that no `X-API-Key` header is ever sent; upstream requests carry
  only `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `awarddelta error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.
- Award-data judgment (from the spec, recorded by the
  coordinator on the firsthand capture read): every tool reads
  PUBLIC federal award data (USAspending). `competitor_watch`
  takes a contractor (company) name and returns that company's
  recent PUBLIC award history plus accrued win/lose/modify
  events — published award history, no score, no verdict, no
  exclusion list, no personal data. `whats_changed_since`'s
  cursor is caller-held (a bookmark the caller passes back);
  the rebuild stores nothing. No payment, metering, contact,
  signup or send tool exists in the surface.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/awarddelta`.
- Package entries: **128**.
- sha256: `fde9edbf9c0831bc2ca2c84fe832421dbb275ba9823e116cbaa1f19dc054a675`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Federal contract delta feed: what changed,
  digests, competitor award watch, award search." — **89 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/awarddelta:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/awarddelta"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/awarddelta`
  and its smoke test asserts the server name `awarddelta`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **56 tests, 56 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 4
   tools, including the no-title and no-required assertions, plus
   session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/govconfeed-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/awarddelta/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/awarddelta/parity-out` → **verdict PASS, parity 100.0**
   (4/4 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
