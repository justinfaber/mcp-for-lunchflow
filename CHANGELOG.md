# Changelog

## 1.3.0

- New tool `lunchflow_get_account_holdings`. It returns investment holdings for accounts whose provider supports them (SnapTrade, MX, Finicity, Pluggy).
  - Largest positions first, by absolute value, so large short positions are kept. Up to 250 rows, fewer if names are long.
  - Only allowlisted fields are returned: name, ticker, ISIN, FIGI, CUSIP, quantity, price, value, cost basis and currency.
  - Security names are cleaned and capped. ISIN, FIGI, CUSIP and ticker must match their exact formats, otherwise they're dropped.
  - Unsupported accounts, including 403 responses and empty lists, get an explanation.
- List results are now cut to a size budget of about 45,000 characters, measured the way Hermes measures them (after its own JSON wrapping). Hermes only previews results over about 50,000 characters, and worst-case text could exceed that before.
  - `truncated: true` can now also mean the size budget was reached, not just the row limit.
  - With `limit` 250, a page typically fits about 175 to 250 transactions, depending on how long the text is.
- Transactions: the shared `currency` is now worked out from all matching transactions, not just the returned page. So a limited page may show per-row currencies where 1.2.0 showed one shared currency. Field names are unchanged.
- Text cleaning:
  - It now also removes lone surrogates and all other default-ignorable characters, such as variation selectors and Hangul fillers. These could hide text, for example a hidden instruction after an emoji.
  - It never cuts an emoji in half when shortening text.
  - Variation selectors are removed outright rather than turned into spaces, so emoji and some Asian-language names keep their normal spacing.
  - Runs of padding and invisible characters are collapsed first. Very long fields are then shortened before Unicode normalisation, so one huge field can no longer use hundreds of MB of memory, and real text after long padding is kept.
- Numbers from the API must be plain decimals: hex, binary, `Infinity` and empty strings are rejected.

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
