# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Lanyardo MCP server, which serves one published dataset for read-only querying. All seven captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `dataset_columns`, `dataset_search`, `dataset_row`, `dataset_compare`, `dataset_stats`, `dataset_top`, `dataset_provenance` — the columns and shape of the dataset, free-text row search, exact-match row lookup, side-by-side comparison across values, summary statistics for a numeric column, ranking by a numeric column, and the dataset's source, licence and citation record.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `datacite error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LANYARDO_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
