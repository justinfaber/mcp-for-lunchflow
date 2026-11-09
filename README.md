# mcp-for-lunchflow

**Unofficial MCP server for Lunch Flow.** Not affiliated with, endorsed by, or supported by Lunch Flow.

A small, read-only [Model Context Protocol](https://modelcontextprotocol.io) server that lets an AI agent read your bank accounts, transactions and balances through the [Lunch Flow Personal API](https://www.lunchflow.app/docs/api/personal-api-overview), using your own API key. It was built for [Hermes Agent](https://hermes-agent.nousresearch.com), but works with any MCP client that runs stdio servers.

- One file of code (`server.mjs`); dependencies are only the official MCP SDK and zod, pinned exactly.
- Read-only: it can only call `GET` endpoints. It cannot move money or change anything.
- Published from GitHub Actions with [npm provenance](https://docs.npmjs.com/generating-provenance-statements), so every release from 1.2.0 onwards is traceable to the commit it was built from.

## Tools

| Tool | What it does |
|---|---|
| `lunchflow_list_accounts` | Lists connected accounts (id, name, institution, provider, currency, status). |
| `lunchflow_get_account_transactions` | Transactions for one account. Optional `from` / `to` (YYYY-MM-DD, inclusive), `include_pending`, and `limit` (default 100, max 250). Newest first. |
| `lunchflow_get_account_balance` | Current balance for one account. |

## Requirements

- Node.js 22 or newer
- A Lunch Flow account with at least one connected bank
- A Lunch Flow API key: in Lunch Flow, go to **Destinations → Add Destination → REST API** and copy the key

## Install

Install a specific version globally. `--ignore-scripts` is a good habit for any package you install as root:

```bash
npm install -g @faberjustin/mcp-for-lunchflow@1.2.0 --ignore-scripts
```

Dependencies are locked with `npm-shrinkwrap.json`, so you get exactly the versions that were tested.

To check the release was built by this repository's GitHub Actions workflow, look at the **Provenance** section on the package's npm page, or verify it from the command line. (`npm audit signatures` does not work on global installs, so this uses a throwaway folder.)

```bash
mkdir -p /tmp/verify && cd /tmp/verify && npm init -y >/dev/null
npm install --ignore-scripts @faberjustin/mcp-for-lunchflow@1.2.0
npm audit signatures
```

## Configure Hermes Agent

1. Store the key in `~/.hermes/.env`, creating the file as private first:

   ```bash
   [ -f ~/.hermes/.env ] || install -m 600 /dev/null ~/.hermes/.env
   chmod 600 ~/.hermes/.env
   echo 'LUNCHFLOW_API_KEY=your-key-here' >> ~/.hermes/.env
   ```

2. Print the config block with the right absolute paths for your machine:

   ```bash
   cat <<EOF

     lunchflow:
       command: "$(command -v node)"
       args: ["$(npm root -g)/@faberjustin/mcp-for-lunchflow/server.mjs"]
       env:
         LUNCHFLOW_API_KEY: "\${LUNCHFLOW_API_KEY}"
       connect_timeout: 30
   EOF
   ```

3. Paste it under `mcp_servers:` in `~/.hermes/config.yaml`, then run `/reload-mcp` and ask: *"list my Lunch Flow accounts"*.

Use absolute paths. Hermes may not use your shell's `PATH`, and may run a bare `node` or `npx` with a different Node.js than the one you installed.

## Configure other MCP clients

Any client that runs stdio servers can use the same command and environment, for example:

```json
{
  "mcpServers": {
    "lunchflow": {
      "command": "mcp-for-lunchflow",
      "env": { "LUNCHFLOW_API_KEY": "your-key-here" }
    }
  }
}
```

That needs both `node` and npm's global `bin` folder on the client's `PATH`, which desktop apps (especially on macOS) often don't have. If the server doesn't start, use absolute paths instead: `"command": "/absolute/path/to/node"` and `"args": ["/absolute/path/to/server.mjs"]`. `command -v node` and `npm root -g` show the paths.

If your client supports remote MCP servers with OAuth, you may prefer Lunch Flow's own hosted server instead; see their [MCP guide](https://www.lunchflow.app/docs/guides/destinations/mcp).

## Configuration

| Variable | Required | Default |
|---|---|---|
| `LUNCHFLOW_API_KEY` | yes | — |
| `LUNCHFLOW_API_URL` | no | `https://www.lunchflow.app/api/v1` |

The server refuses to start if `NODE_OPTIONS` is set, because runtime flags could weaken it. Some setups set it globally, for example `--use-openssl-ca` behind a corporate proxy. If yours does, unset it for this server only: MCP clients pass only the environment you configure, plus a few basics.

## Security

What the server does:

- **Your key stays private.** It is sent only in the `x-api-key` header to the configured API host, and never logged or returned to the model.
- **The connection can't be weakened.** TLS is required. The server refuses to start with an invalid key, credentials in the URL, `NODE_TLS_REJECT_UNAUTHORIZED=0`, or `NODE_OPTIONS` set.
- **Redirects are refused,** so the key is never forwarded to another host.
- **Requests are bounded.** Each one has a hard 20-second deadline and a 5 MB response cap.
- **Output is cleaned.** Only allowlisted fields are returned. Text from banks and third parties (merchant names, descriptions, account names) is length-capped and stripped of control, invisible and text-direction characters.
- **Tool input is strict.** Account ids, dates and limits are validated, and unknown arguments are rejected.

What it cannot do for you:

- **Transaction text is written by other people.** Anyone who pays you, or any merchant, can put text into your transactions. An AI agent might treat that text as instructions. The server cleans and caps it, but it cannot rule this out. Don't let an agent that reads your bank data also act without your approval. In particular, avoid running it as root with an unrestricted shell.
- **An agent with a shell can read the key** from `~/.hermes/.env` or the process environment. The Lunch Flow key is read-only, but it can see all your bank data. Rotate it if you suspect it was exposed.

To report a vulnerability, see [SECURITY.md](SECURITY.md).

## Development

```bash
npm ci --ignore-scripts
npm test
```

The tests start the real server over stdio against a local mock of the Lunch Flow API.

## Disclaimer

This is an independent, unofficial project, provided as-is and maintained on a best-effort basis. "Lunch Flow" is used only to describe what this software works with. All trademarks belong to their respective owners.

## License

[MIT](LICENSE) © 2026 Justin Faber
