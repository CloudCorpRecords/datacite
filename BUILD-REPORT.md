# BUILD REPORT — datacite

Clean-room rebuild of `com.lanyardo/site` v1.0.1
(remote-only MCP server at `https://lanyardo.com/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/datacite`
0.1.0, registry name `io.github.CloudCorpRecords/datacite`.

## Surface built

**FULL surface: all 7 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `dataset_columns` | *(none — no `required` key in the capture, served verbatim)* |
| `dataset_search` | `query` |
| `dataset_row` | `column`, `value` |
| `dataset_compare` | `column`, `values` |
| `dataset_stats` | `column` |
| `dataset_top` | `column` |
| `dataset_provenance` | *(none — no `required` key in the capture, served verbatim)* |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, with the `inputSchema.$schema` marker
(`http://json-schema.org/draft-07/schema#`) exactly as captured.
`dataset_search` / `dataset_row` / `dataset_compare` /
`dataset_stats` / `dataset_top` carry `additionalProperties: false`
exactly as captured; `dataset_columns` and `dataset_provenance`
carry no `additionalProperties` key — none was captured, none was
invented. Property shapes are served verbatim, including
`dataset_search`'s optional `limit` (integer, 1..50),
`dataset_compare`'s `values` (array of minLength-1 strings,
2..10 items), and `dataset_top`'s optional `limit` (integer,
1..50) and `ascending` (boolean, with its captured description).
The tool literals were generated programmatically from
`harness/fixtures/lanyardo-tools-original.json` by the script
at `/tmp/laneb7-datacite-scratch/gen.py` (parse + re-serialise,
spliced into `src/forward.ts` between the TOOL_DEFS_GENERATED
markers) — never retyped — and the generator's round-trip check
confirmed the spliced block parses back deep-equal to the
fixture's 7 tool objects, in order. Tools are registered via
the MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

The surface is read-only querying of one published dataset:
its columns and provenance banner, full-text row search,
exact-match rows, multi-value comparison, numeric column
statistics, top/lowest rows, and the dataset's
source/licence/citation record. No sends, no payments, no
caller-named person/entity screening.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, the `required` arrays (`query`;
`column`+`value`; `column`+`values`; `column`; `column`; and
none for `dataset_columns` / `dataset_provenance`), and the
property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://lanyardo.com/mcp`.
- Env override `LANYARDO_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `LANYARDO_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` /
  `LANYARDO_TOKEN` variables present.
- Error rendering `datacite error:` text for upstream failures
  — labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Deviations

**None.** No write-guard workaround was needed in this session:
the lockfile, `npm ci`, `tsc` build and test suite all ran
through the normal routes, and `dist/` is the direct output of
`tsc` on the validated sources.

## Lockfile

Generated clean from birth (Route 3, clean dir, no
`node_modules`, no pre-existing lockfile):
`npm install --package-lock-only --ignore-scripts --offline`
→ success. Parses as JSON; `name` == `@reneturcios/datacite`;
exactly **128** package entries; sha256
`8caee476c1a25724e627828e086212875538a767fbc9f55efe48a52c566eebd0`.
`npm ci --ignore-scripts --offline` then succeeded and the
lockfile sha256 was re-verified **unchanged** afterwards.

## server.json

- Description exactly: "Query a published dataset: columns, search, exact rows, comparisons, stats, top rows, provenance." — **97 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/datacite:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/datacite"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/datacite`
  and its smoke test asserts the server name `datacite`.

## Verification (run 2026-10-28, builder session)

1. `npm install --package-lock-only --ignore-scripts --offline`
   → success; lockfile parses, name `@reneturcios/datacite`,
   128 package entries (see Lockfile).
2. `npm ci --ignore-scripts --offline` → success; lockfile
   sha256 unchanged.
3. `tsc --noEmit` (direct `node_modules/.bin/tsc`) → exit 0.
4. `tsc` build → exit 0; `node dist/cli.js --version` → `0.1.0`.
5. Full suite (`tsx --test test/*.test.ts`) → **70 tests,
   70 pass, 0 fail** (registration-integrity suite deep-equals
   the served tools/list against the fixture's 7 tools, in
   order, at both the registry level and through an
   in-memory-transport MCP `tools/list`; plus per-tool schema
   detail assertions, session handling/reuse, env override,
   the no-Authorization-header assertions under every
   environment, verbatim argument forwarding for all 7 tools
   and result pass-through including upstream isError, SSE
   parsing, error rendering, MCP-layer validation, and CLI
   behaviour).
6. Parity (harness, tools-file mode):
   `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/lanyardo-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/datacite/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/datacite/parity-out`
   → **verdict PASS, parity 100.0** (7/7 identical, 0 missing,
   0 extra, no schema diffs). Output in `parity-out/`.
