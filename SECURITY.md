# Security policy

## Reporting a vulnerability

Please report security problems privately. Do not open a public issue.

Use GitHub's private vulnerability reporting: the **Security** tab of this repository, then **Report a vulnerability**.

Please include what you found, how to reproduce it, and the version you tested. You can expect an acknowledgement within a week. This is a personal, best-effort project, so fix times may vary.

## Supported versions

Only the latest release receives fixes.

## Scope

In scope: this server's code, its published npm package, and its release workflow. Examples include key leakage, ways around the TLS, redirect, size or time limits, unsafe handling of API responses, and input validation bypasses.

Out of scope:
- Lunch Flow's own services. Report those to Lunch Flow.
- The MCP client or agent running this server.
- The general risk that third-party transaction text can influence an AI agent, which is documented in the README. A concrete bypass of the server's text cleaning is in scope.

## Verifying releases

Releases are published only from this repository's GitHub Actions workflow, using npm trusted publishing with provenance. You can verify a release on the package's npm page (Provenance section), or with `npm audit signatures` in a project folder where it is installed locally. That command doesn't support global installs.
