# SkillScan Rules Changelog

## 2026.04.29.1

Pattern update 2026-04-29. Three new PSV rules covering two unpatched VS Code extension vulnerabilities and a critical Azure DevOps MCP Server missing-authentication flaw. Vuln DB additions for two new products.

- **PSV-051** (high, new): **Code Runner VS Code extension RCE via malicious executorMap settings (CVE-2025-65715, formulahendry.code-runner, CVSS 7.8)** — OX Security disclosed in February 2026 that the Code Runner extension (formulahendry.code-runner, 37M+ installs) passes code-runner.executorMap settings directly to Node.js child_process.spawn() with shell:true without sanitization. An attacker can craft a workspace with a malicious executorMap entry (e.g., a reverse shell via /dev/tcp) that executes on every Run Code invocation. Affects all versions through v0.12.2. No official patch available. Mitigate by auditing settings.json for unexpected executorMap entries or removing the extension.
- **PSV-052** (critical, new): **Live Server VS Code extension local file exfiltration via CORS bypass (CVE-2025-65717, ritwickdey.liveserver, CVSS 9.1)** — OX Security disclosed that the Live Server extension (ritwickdey.liveserver, 72M+ installs) serves workspace files over localhost:5500 without any CORS restrictions. When a developer visits a malicious webpage while Live Server is running, embedded JavaScript can recursively crawl http://localhost:5500 and exfiltrate all served files to an attacker-controlled server. Affects v5.7.9 and all earlier versions. No official patch available. Mitigate by stopping Live Server when browsing untrusted sites.
- **PSV-053** (critical, new): **Azure DevOps MCP Server missing authentication information disclosure (CVE-2026-32211, @azure-devops/mcp, CVSS 9.1)** — Microsoft disclosed on 2026-04-03 that the Azure DevOps MCP Server (@azure-devops/mcp npm package) exposes MCP tools for Azure DevOps work items, repositories, pipelines, and pull requests without enforcing authentication (CWE-306). Any unauthenticated network attacker can access sensitive project data. No official patch available as of 2026-04-29. Mitigate by placing the server behind an authenticated reverse proxy and rotating exposed credentials.
- **Vuln DB additions:** `code-runner` all versions (CVE-2025-65715); `liveserver` all versions (CVE-2025-65717).

Total: 245 static rules + 17 chain rules.

Sources:
- CVE-2025-65715 (Code Runner RCE): https://thehackernews.com/2026/02/critical-flaws-found-in-four-vs-code.html
- CVE-2025-65715 (OX Security): https://www.ox.security/blog/cve-2025-65715-code-runner-vscode-rce/
- CVE-2025-65717 (Live Server): https://thehackernews.com/2026/02/critical-flaws-found-in-four-vs-code.html
- CVE-2025-65717 (OX Security): https://www.ox.security/blog/cve-2025-65717-live-server-vscode-vulnerability/
- CVE-2026-32211 (Azure DevOps MCP): https://cvefeed.io/vuln/detail/CVE-2026-32211
- CVE-2026-32211 (WindowsForum): https://windowsforum.com/threads/cve-2026-32211-azure-mcp-server-auth-flaw-leaks-info-cvss-9-1.409622/

## 2026.04.27.1

Pattern update 2026-04-27. Three new rules (MAL-073, PSV-045, PSV-046) covering a VS Code cryptojacking campaign, an unpatched VS Code extension RCE, and a LibreChat MCP STDIO privilege escalation. IOC enrichment with one new C2 domain; vuln DB additions for two products.

- **MAL-073** (high, new): **Mark H VS Code extension cryptojacking campaign (asdf11.xyz C2, XMRig miner, April 2026)** — In April 2026 a threat actor using the Marketplace publisher name "Mark H" published nine trojanized extensions (MarkH.discord-rich-presence-vs, MarkH.golang-compiler-vscode, MarkH.html-obfuscator-vscode, MarkH.python-obfuscator-vscode, MarkH.rust-compiler-vs, MarkH.claude-ai, MarkH.chatgpt-agent-vscode) that accumulated over 1 million combined installs in three days. All extensions contact C2 domain asdf11[.]xyz, drop an XMRig cryptocurrency miner, and persist via a Windows scheduled task named "OnedriveStartup". Microsoft removed the extensions from the Marketplace.
- **PSV-045** (high, new): **Markdown Preview Enhanced RCE via crafted markdown file (CVE-2025-65716, shd101wyy.markdown-preview-enhanced, CVSS 8.8)** — OX Security disclosed in February 2026 that a crafted .md file triggers JavaScript execution inside Markdown Preview Enhanced's VS Code preview pane (shd101wyy.markdown-preview-enhanced, 4M+ installs, all versions ≤ 0.8.18). The script can enumerate open local ports and exfiltrate workspace file contents to an attacker server. No official patch is available. Mitigate by disabling script execution in the extension's settings.
- **PSV-046** (critical, new): **LibreChat MCP STDIO unauthorized root command execution (CVE-2026-22252, < 0.8.2-rc2, CWE-285)** — The MCP STDIO interface in LibreChat < 0.8.2-rc2 fails to enforce authorization checks on the command execution path, allowing any authenticated user to execute arbitrary shell commands as the root user inside the container via a single API request. Fixed in 0.8.2-rc2. Upgrade immediately; also run the container as a non-root user as defense-in-depth.
- **IOC additions:** C2 domain `asdf11.xyz` (Mark H VS Code cryptojacking campaign).
- **Vuln DB additions:** `markdown-preview-enhanced` all versions (CVE-2025-65716); `librechat` < 0.8.2-rc2 (CVE-2026-22252).

Total: 239 static rules + 14 chain rules.

Sources:
- Mark H VS Code cryptojacking: https://thehackernews.com/2026/04/malicious-vs-code-extensions-fuel-large-scale-cryptojacking.html
- Mark H cryptojacking (UnderCode): https://undercodenews.com/malicious-vs-code-extensions-fuel-large-scale-cryptojacking-operation/
- CVE-2025-65716 (Markdown Preview Enhanced): https://thehackernews.com/2026/02/critical-flaws-found-in-four-vs-code.html
- CVE-2025-65716 (OX Security): https://www.ox.security/blog/cve-2025-65716-markdown-preview-enhanced-vscode-vulnerability/
- CVE-2026-22252 (LibreChat MCP STDIO RCE): https://thehackernews.com/2026/04/anthropic-mcp-design-vulnerability.html

## 2026.04.26.3

Hotfix: add missing `metadata:` blocks to MAL-071, MAL-072, SE-006 (added in PR #11 earlier today). Without metadata, every rule fails the `test_rule_metadata_guard.py` check that runs in `skillscan-security` CI on every bundled-snapshot sync — the previous commit broke the sync flow. The rules-side `validate.yml` did not catch this because metadata isn't in its required-fields set; consider adding a `metadata` requirement in a follow-up.

Also moved tag-like strings (`tradingclaw`, `needle-stealer`, `amos-stealer`, `cursor-ai-agent`, `clickfix`, `athr`, `vishing`, `toad`) from each rule's top-level `references:` list into `metadata.tags` where they belong; only HTTP URLs remain in `metadata.references`. Pattern, mitigation, and test_input/test_expect are unchanged for all three rules.

No new detection content. No vuln DB or IOC changes.

## 2026.04.26.1

Pattern update 2026-04-26. Two new rules: one passive surveillance CVE (MCPJam Inspector unauthenticated RCE) and one supply chain campaign (Contagious Interview cross-ecosystem April 2026 wave). Plus IOC enrichment with three Contagious Interview C2 domains, one IP, and one C2 URL. Vuln DB additions for MCPJam Inspector and eleven Contagious Interview npm/PyPI packages.

- **PSV-044** (critical, new): **MCPJam Inspector RCE via unauthenticated /api/mcp/connect (CVE-2026-23744, GHSA-232v-j27c-5pp6, @mcpjam/inspector <= 1.4.2)** — Critical RCE (CVSS 9.8, CWE-306) in MCPJam Inspector versions <= 1.4.2. The `/api/mcp/connect` HTTP endpoint accepts `command` and `args` parameters and executes them directly without authentication or validation. Because the tool binds to `0.0.0.0` by default, any network-reachable attacker can achieve arbitrary command execution with a single crafted HTTP POST — no privileges or user interaction required. Fixed in version 1.4.3.
- **SUP-046** (critical, new): **Contagious Interview cross-ecosystem supply chain — April 2026 wave (DPRK/UNC1069, npm/PyPI/Go/Rust)** — The North Korean threat cluster UNC1069 (Contagious Interview / Sapphire Sleet / Stardust Chollima) has published 1,700+ malicious packages across five package ecosystems since January 2025. The April 2026 wave includes npm packages `dev-log-core`, `logger-base`, `logkitx`, `pino-debugger`, `debug-fmt`, `debug-glitz` and PyPI packages `logutilkit`, `apachelicense`, `fluxhttp`, `license-utils-kit`. All packages POST to `https://apachelicense.vercel.app/getAddress?platform=<platform>`, download `ecw_update.zip`, and extract to the hardcoded temp dir `410BB449A-72C6-4500-9765-ACD04JBV827V32V`, then execute a platform-specific cross-platform RAT/infostealer. Staging: `logkit.onrender.com`, `logkit-tau.vercel.app`.
- **IOC additions:** Contagious Interview C2 domains `apachelicense.vercel.app`, `logkit.onrender.com`, `logkit-tau.vercel.app`; IP `66.45.225.94`; URL `https://apachelicense.vercel.app/getAddress`.
- **Vuln DB additions:** `@mcpjam/inspector` 1.4.0–1.4.2 (CVE-2026-23744); npm backdoors `dev-log-core`, `logger-base`, `logkitx`, `pino-debugger`, `debug-fmt`, `debug-glitz` (CONTAGIOUS-INTERVIEW-NPM-2026-0408, all versions); PyPI backdoors `logutilkit`, `apachelicense`, `fluxhttp`, `license-utils-kit` (CONTAGIOUS-INTERVIEW-PYPI-2026-0408, all versions).

Note: CHANGELOG entry for rulepack `2026.04.25.3` is missing — the version was bumped in default.yaml but no corresponding `## 2026.04.25.3` section was written. Flagged for separate cleanup PR.

Total: 233 static rules + 14 chain rules.

Sources:
- CVE-2026-23744 (MCPJam Inspector): https://github.com/advisories/GHSA-232v-j27c-5pp6
- CVE-2026-23744 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-23744
- CVE-2026-23744 (PoC): https://github.com/boroeurnprach/CVE-2026-23744-PoC
- Contagious Interview cross-ecosystem (Socket): https://socket.dev/blog/contagious-interview-campaign-spreads-across-5-ecosystems
- Contagious Interview 1,700 packages (THN): https://thehackernews.com/2026/04/n-korean-hackers-spread-1700-malicious.html

## 2026.04.25.4

Pattern update 2026-04-25 (4th of the day). Three new rules across supply chain, passive surveillance, and prompt injection — all anchored to disclosed CVEs / GHSAs / first-party threat reports from the past two weeks. Plus IOC enrichment for two recent campaigns (GPT-Proxy + Sandworm_Mode) and vuln-DB entries for the affected packages.

- **SUP-045** (critical, new): **GPT-Proxy backdoor — `kube-health-tools` (npm) / `kube-node-health` (PyPI) Chinese LLM relay (April 22, 2026)** — Aikido Security disclosed twin packages on npm and PyPI that install a two-stage Zig-compiled binary masquerading as `node-health-check --mode=daemon`. The dropper fetches stage 2 from `github.com/gibunxi4201/kube-node-diag/releases/download/v2.0/`, writes to `/tmp/.kh`, and proxies attacker OpenAI-compatible API calls (`/v1/chat/completions`, `/v1/completions`, `/v1/models`) through the victim host as a Chinese LLM relay. C2: `sync.geeker.indevs.in`. No CVE assigned.
- **PSV-043** (high, new): **Azure Data Explorer MCP Server KQL injection — CVE-2026-33980, GHSA-vphc-468g-8rfp, <= 0.1.1** — KQL injection in Microsoft's `azure-data-explorer-mcp-server` <= 0.1.1 (CVSS 8.3 HIGH). The MCP tool handlers `get_table_schema`, `sample_table_data`, and `get_table_details` interpolate the user-supplied `table_name` parameter into KQL queries via Python f-strings, bypassing parameterization. Fixed past commit `0abe0ee55279e111281076393e5e966335fffd30`.
- **PINJ-024** (critical, new): **Windsurf MCP STDIO command injection — CVE-2026-30615, Windsurf 1.9544.26** — OX Security disclosed April 15, 2026 that attacker-controlled HTML rendered by Windsurf 1.9544.26 (CVSS 8.0 HIGH) can carry prompt-injection instructions that automatically register a malicious MCP STDIO server, achieving zero-click RCE. Root cause is the MCP STDIO transport executing arbitrary commands as subprocesses regardless of whether they implement the protocol.
- **IOC additions:** GPT-Proxy C2 domains `sync.geeker.indevs.in`, `geeker.indevs.in`. Sandworm_Mode npm worm (Socket, Feb 20, 2026) C2 + DNS-tunneling domains `pkg-metrics.official334.workers.dev`, `freefan.net`, `fanfree.net`.
- **Vuln DB additions:** `azure-data-explorer-mcp-server` 0.1.0–0.1.1 (CVE-2026-33980), `windsurf` 1.9544.26 (CVE-2026-30615), `kube-health-tools` (npm, all versions, NPM-BACKDOOR-2026-0422), `kube-node-health` (PyPI, all versions, PYPI-BACKDOOR-2026-0422), Sandworm_Mode npm typosquats `claud-code` / `cloude-code` / `suport-color` / `rimarf` / `yarsg` (NPM-SANDWORM-MODE-2026-0220).

Total: 231 static rules + 14 chain rules.

Sources:
- GPT-Proxy backdoor (kube-health-tools / kube-node-health): https://www.aikido.dev/blog/gpt-proxy-backdoor-npm-pypi-chinese-llm-relay
- CVE-2026-33980 (Azure Data Explorer MCP): https://nvd.nist.gov/vuln/detail/CVE-2026-33980
- CVE-2026-33980 (GHSA): https://github.com/Azure/azure-data-explorer-mcp-server/security/advisories/GHSA-vphc-468g-8rfp
- CVE-2026-33980 (SentinelOne): https://www.sentinelone.com/vulnerability-database/cve-2026-33980/
- CVE-2026-30615 (Windsurf MCP STDIO): https://nvd.nist.gov/vuln/detail/CVE-2026-30615
- CVE-2026-30615 (OX Security advisory): https://www.ox.security/blog/mcp-supply-chain-advisory-rce-vulnerabilities-across-the-ai-ecosystem/
- Sandworm_Mode npm worm (Socket): https://socket.dev/blog/sandworm-mode-npm-worm-ai-toolchain-poisoning

## 2026.04.25.2

Pattern update 2026-04-25. One new PSV rule (MCP Ruby SDK session fixation) and three new vuln DB entries.

- **PSV-041** (high, new): **MCP Ruby SDK SSE session hijacking via session fixation** — CVE-2026-33946 (CVSS 7.5, CWE-384 Session Fixation / CWE-639 Authorization Bypass via User-Controlled Key) in the `mcp` Ruby gem (modelcontextprotocol/ruby-sdk) < 0.9.2. The Streamable HTTP transport (`streamable_http_transport.rb`) does not invalidate or regenerate session IDs when SSE connections are established, allowing an attacker who obtains a valid session ID to replay it and fully hijack the victim's SSE stream, intercepting all real-time MCP tool results. Upgrade to `mcp >= 0.9.2`.
- **Vuln DB additions:** `mcp-ruby-sdk` 0.9.0–0.9.1 (CVE-2026-33946, Ruby gem `mcp`), `codebase-mcp` rolling (CVE-2026-5023, no patch), `@nor2/heim-mcp` 0.1.0–0.1.3 (CVE-2026-5602).

Total: 225 static rules + 14 chain rules.

Sources:
- CVE-2026-33946 (MCP Ruby SDK): https://rubysec.com/advisories/CVE-2026-33946/
- CVE-2026-33946 (GitLab advisory): https://advisories.gitlab.com/pkg/gem/mcp/CVE-2026-33946/
- CVE-2026-5023 (codebase-mcp): https://radar.offseq.com/threat/cve-2026-5023-os-command-injection-in-dedeveloper2-cc03c992
- CVE-2026-5602 (@nor2/heim-mcp): https://vuldb.com/vuln/355394

## 2026.04.25.1

Pattern update 2026-04-25. Two new rules: one supply chain (TeamPCP Bitwarden CLI worm) and one passive surveillance CVE (Azure DevOps MCP missing auth).

- **PSV-040** (critical, new): **Azure DevOps MCP Server missing authentication** — CVE-2026-32211 (CVSS 9.1, CWE-306) in `@azure-devops/mcp`. Microsoft disclosed April 3, 2026. The server lacks any authentication layer, allowing unauthenticated network attackers to read Azure DevOps work items, repos, pipelines, and pull requests. No patch available as of 2026-04-25; mitigate with network-level access controls.
- **SUP-042** (critical, new): **Shai-Hulud @bitwarden/cli@2026.4.0 supply chain compromise (TeamPCP)** — `@bitwarden/cli` version `2026.4.0` was compromised in a TeamPCP supply chain attack on April 22, 2026. A preinstall hook runs `bw_setup.js`, bootstraps Bun, and executes `bw1.js`, a self-propagating npm worm and credential harvester that exfiltrates SSH keys, cloud secrets, AI tool configs (.claude, .cursor, .windsurf), and crypto wallets to `audit.checkmarx.cx/v1/telemetry` (a typosquat of the real Checkmarx domain). The worm re-injects itself into all npm packages the victim token can publish.
- **IOC additions:** domain `audit.checkmarx.cx`, URL `https://audit.checkmarx.cx/v1/telemetry`.
- **Vuln DB additions:** `@bitwarden/cli` 2026.4.0 (SUP-042), `@azure-devops/mcp` 2.5.0 (CVE-2026-32211).

Total: 224 static rules + 14 chain rules.

Sources:
- CVE-2026-32211 (Azure DevOps MCP): https://windowsnews.ai/article/cve-2026-32211-critical-azure-mcp-server-authentication-flaw-exposes-sensitive-data-cvss-91.409622
- Shai-Hulud Bitwarden CLI: https://www.aikido.dev/blog/shai-hulud-npm-bitwarden-cli-compromise
- Shai-Hulud Bitwarden CLI (OX): https://www.ox.security/blog/shai-hulud-bitwarden-cli-supply-chain-attack/
- Shai-Hulud Bitwarden CLI (JFrog): https://research.jfrog.com/post/bitwarden-cli-hijack/

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
