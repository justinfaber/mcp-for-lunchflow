# Changelog

## 1.2.0 (first public release)

- Three read-only tools: `lunchflow_list_accounts`, `lunchflow_get_account_transactions`, `lunchflow_get_account_balance`.
- Transactions are sorted newest first, with optional `from` / `to` / `include_pending`, and `limit` (default 100, max 250). The currency is listed once when all rows share it.
- Hardening:
  - TLS is required, and redirects are refused.
  - Requests have a 20 s deadline and a 5 MB response cap, and the connection is closed when either is hit.
  - Output fields are allowlisted, and third-party text is cleaned and capped.
  - Tool input is validated strictly.
  - The server refuses to start with unsafe configuration.
- Requires Node.js 22 or newer.
- Dependencies (MCP SDK 1.32.2, zod 4.6.7) are pinned with `npm-shrinkwrap.json`.
