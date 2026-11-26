# datacite

Ask questions of a published dataset without downloading it, cleaning it, or keeping a local copy in sync. datacite is a small forwarding client for the hosted Lanyardo MCP service, which serves one published dataset for read-only querying: the tool list is answered from your own machine using captured definitions, and each call travels upstream, with the service's answer handed back exactly as it arrived.

All seven of the service's captured tools are exposed. Start with the dataset's shape — its columns, which of them are numeric, its row count and its provenance banner — then search rows by free text, fetch the rows where a column equals a value exactly, line rows up side by side across several values for "X vs Y" questions, pull summary statistics for a numeric column, rank rows highest or lowest by a numeric column, and read the provenance record so any figure you quote can be attributed to its source, date, licence and citation.

## The tools

- **`dataset_columns`** — The columns of the dataset, which of them are numeric, the row count and the provenance banner. No arguments. Call this first to learn the schema before querying.
- **`dataset_search`** — Rows whose cells contain the `query` text, matched case-insensitively across every cell, up to 50 rows. An optional `limit` (1–50) narrows the result.
- **`dataset_row`** — The rows where a named `column` equals a `value` exactly, case-insensitively. For precise lookups once you know the column and the key.
- **`dataset_compare`** — The rows whose `column` is any of the given `values` (2–10 of them), returned in the order the values were given. Built for side-by-side "X vs Y" comparisons.
- **`dataset_stats`** — Count, minimum, maximum, mean, median and sum of a numeric `column`. Grouping commas and currency marks are handled; non-numeric rows are excluded and counted.
- **`dataset_top`** — The highest rows by a numeric `column`, or the lowest when `ascending` is true. An optional `limit` (1–50) sets how many rows come back. For "which is the most/least X" questions.
- **`dataset_provenance`** — The dataset's source, the date it was computed, its licence and its citation. No arguments. Read this to attribute a figure correctly.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/datacite mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "datacite": {
      "command": "npx",
      "args": ["-y", "@reneturcios/datacite", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/datacite
docker run --rm ghcr.io/cloudcorprecords/datacite dataset_search '{"query":"alpha"}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
datacite dataset_columns '{}'
datacite dataset_search '{"query":"alpha","limit":10}'
datacite dataset_row '{"column":"name","value":"alpha"}'
datacite dataset_compare '{"column":"name","values":["alpha","beta"]}'
datacite dataset_stats '{"column":"score"}'
datacite dataset_top '{"column":"score","limit":5}'
datacite dataset_provenance '{}'
datacite --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `LANYARDO_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://lanyardo.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The Lanyardo registry record lists no headers for its remote, so datacite never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `datacite error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, datacite repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged for every one of the seven tools.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
