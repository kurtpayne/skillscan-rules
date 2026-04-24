# SkillScan Rules Changelog

## 2026.04.24.2

Second pattern update of 2026-04-24. Ports one previously-stranded rule from `skillscan-security` and adds two new CVE-backed detections.

- **PSV-036** (high, ported from `skillscan-security`): **MCP server command injection via insufficient shellQuote** — CVE-2026-5603 in `@elgentos/magento2-dev-mcp <= 1.0.2`. All 16 MCP tools concatenate user input into shell commands escaped only with `shell-quote`'s single-quote style; on Windows, `cmd.exe` doesn't treat single quotes as quoting characters, so metacharacters (`&`, `|`, `>`, `<`) still break out. Fixed in 1.0.3.
- **PSV-038** (high, new): **Flowise CSV Agent prompt-injection RCE** — CVE-2026-41264 / GHSA-3hjv-c53m-58jj (CVSS 8.1, CWE-184) in `flowise-components < 3.1.0`. The CSV Agent node's `run` method sends user prompts to an LLM that emits pandas operations executed via Pyodide; the import-allowlist regex is bypassable by `import os as pandas` style aliasing. Unauthenticated. Fixed in 3.1.0.
- **PSV-039** (medium, new): **Splunk MCP Server session-token log disclosure** — CVE-2026-20205 (CVSS 7.2, CWE-532) in Splunk MCP Server < 1.0.3. Session and authorization tokens are written to the `_internal` index in cleartext; anyone with `mcp_tool_admin` or `_internal` read rights can harvest them. Fixed in 1.0.3.
- **Taxonomy rename:** all PSV rules' `category`, tags, and technique-name fields migrated from `platform_security` / `Platform security vulnerability` to `passive_surveillance` / `Passive surveillance vulnerability` to match the authoritative display mapping in `sync-website-rules.py`. No pattern changes; purely metadata.
- **Vuln DB additions:** `@elgentos/magento2-dev-mcp` 1.0.0–1.0.2 (CVE-2026-5603), `flowise-components` 3.0.0–3.0.5 (CVE-2026-41264), `splunk-mcp-server` 1.0.0–1.0.2 (CVE-2026-20205).

Total: 222 static rules + 14 chain rules.

Sources:
- CVE-2026-5603 (magento2-dev-mcp): https://dev.to/armor1ai/how-to-check-your-mcp-server-for-cve-2026-5603s-vulnerability-pattern-and-why-shellquote-isnt-gdn
- CVE-2026-41264 (Flowise CSV Agent): https://advisories.gitlab.com/npm/flowise-components/CVE-2026-41264/
- CVE-2026-20205 (Splunk MCP Server): https://www.sentinelone.com/vulnerability-database/cve-2026-20205/

## 2026.04.18.1

Initial release — migrated from skillscan-security repo.

- 220+ static rules across 12 categories
- 14 chain rules
- 17 multilang rules
- 5,500+ IOC entries
- 60+ vulnerable package versions
