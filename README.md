# VulnTracker MCP server

Hosted [MCP](https://modelcontextprotocol.io) server for coding agents: ask which of your
dependencies are vulnerable, which CVEs affect an exact version, and which version fixes each.
Backed by VulnTracker's affected-version ledger, not by a model's memory.

This repository holds the connection instructions and the registry manifest. The server itself
runs at `https://api.vulntracker.io/mcp`; there is nothing to install or run locally.

- **Endpoint:** `https://api.vulntracker.io/mcp` (Streamable HTTP)
- **Auth:** `Authorization: Bearer <API key>`, keys start with `vt_`
- **Plans:** works with a free VulnTracker account (30 read-only tool calls a day). Pro (1,000/day)
  and Business (5,000/day) add the board and tracking tools. Quota is per account and resets 00:00 UTC.

<!-- demo.gif: "check this repo with VulnTracker" in Claude Code, then "add the exploited ones to the board" -->

## Connect

Create a key at **Settings → API keys** on [vulntracker.io](https://vulntracker.io). It is shown once.

**Claude Code**

```bash
claude mcp add --transport http vulntracker https://api.vulntracker.io/mcp \
  --header "Authorization: Bearer vt_…"
```

**Cursor** (`.cursor/mcp.json`) and **Claude Desktop** (`claude_desktop_config.json`)

```json
{
  "mcpServers": {
    "vulntracker": {
      "url": "https://api.vulntracker.io/mcp",
      "headers": { "Authorization": "Bearer vt_…" }
    }
  }
}
```

**Codex** (`~/.codex/config.toml`)

```toml
[mcp_servers.vulntracker]
url = "https://api.vulntracker.io/mcp"
http_headers = { Authorization = "Bearer vt_…" }
```

## Try it

With the server connected, say:

> check this repo for vulnerabilities with VulnTracker

The agent reads your lock files, calls `check_dependencies` with the resolved versions, verifies
the hits with `lookup_cve` and reports a table with fix versions. Then:

> add the exploited ones to the board and track these dependencies

(Pro / Business). In Claude Code the same procedure is available as the
`/mcp__vulntracker__scan_dependencies` prompt.

## Tools

| tool | question it answers |
|---|---|
| `check_dependencies` | Which of these `name@version` pairs are vulnerable? Up to 50 per call |
| `search_products` | What is the VulnTracker slug for `openssl` / `log4j` / `fortios`? |
| `product_versions` | Which versions of this product are known, with CVE and fix counts? |
| `check_version` | Which CVEs affect version X of this product, and which version fixes each? |
| `product_cves` | All CVEs recorded against a product, with the version ranges they name |
| `lookup_cve` | Full record: scores, EPSS, KEV / SSVC, affected products and fixes, vendor advisories, exploit repos, remediation guide |
| `search_cves` | Filtered list: vendor, product, severity, exploited, remediation kind, dates |
| `remediation_guide` | VulnTracker's published fix / workaround guide for a CVE |
| `list_board_cards` | Your team's CVE board with triage status and notes |
| `list_trackings` | What your team is watching |
| `add_to_board` | *write, Pro+* — put CVEs on the team board with a status and a note |
| `update_board_card` | *write, Pro+* — change a card's status, resolution or notes |
| `track` / `untrack` | *write, Pro+* — start or stop watching a product, vendor, CVE or keyword |

Write tools act on the team the API key belongs to and name it in the answer. Agents are
instructed to write only when asked.

## Reading the answers

- `fix_stated_by_source=false`: the fix version was derived from a range bound, not stated by the vendor.
- `match_confidence`: `token` and `exact` are reliable; `fallback` items carry `needs_confirmation=true`
  and are left out of the vulnerable count until confirmed against `matched_vendor`.
- `all_versions_cves`: the source named no version at all. `shadowed_range_cves`: an open-start
  `< X` entry next to an `A..X` entry with the same end; probably the same branch restated.
- `version_scheme="git"`: the version is a commit id and cannot be compared with a release number.

## Errors

| | |
|---|---|
| HTTP 401 | malformed, unknown or revoked key |
| tool error `authentication required` | tool call without a key; tool listing works without one |
| tool error `plan_required` | a write tool on the free plan |
| tool error `daily quota exceeded` | wait for 00:00 UTC or upgrade; a new key does not reset it |

## Links

- Documentation: https://vulntracker.io/docs/mcp
- Pricing: https://vulntracker.io/pricing
- Issues and questions: open an issue here
