# SkillScan Rules Changelog

## 2026.05.03.1

Pattern update 2026-05-03. One new PSV rule covering the MCP TypeScript SDK cross-client data leak (CVE-2026-25536). Vuln DB addition for @modelcontextprotocol/sdk affected version range.

- **PSV-063** (high, new): **MCP TypeScript SDK cross-client data leak via shared transport instance reuse (CVE-2026-25536, GHSA-345p-7cg4-v4c7, CVSS 7.1, CWE-362, @modelcontextprotocol/sdk 1.10.0–1.25.3)** — When a single `McpServer` or `StreamableHTTPServerTransport` instance is reused across multiple concurrent client connections, JSON-RPC message ID collisions cause responses to route to the wrong client, leaking tool execution results, resource data, authentication context, and prompt responses across session boundaries. In multi-tenant or stateless deployments, an attacker sending overlapping requests can receive another client's data without additional privileges (network, low-privilege required). Patched in v1.26.0 which adds runtime guards converting silent misrouting into immediate errors. Upgrade `@modelcontextprotocol/sdk` to ≥ 1.26.0 and ensure fresh `McpServer` and transport instances per request (stateless) or session (stateful). Also affects `mcp-handler` (Vercel) < 1.1.0 via peer dependency (GHSA-w2fm-25vw-vh7f).
- **Vuln DB additions:** `npm/@modelcontextprotocol/sdk` version range ≥1.10.0, ≤1.25.3 → CVE-2026-25536 (high, CVSS 7.1, fixed: 1.26.0).

Candidates researched and already covered or lacking hard anchors: PyTorch Lightning Mini Shai-Hulud (SUP-048 + MAL entry, 2026.05.01.1), SAP CAP npm Mini Shai-Hulud (existing rule), CanisterSprawl worm OpenWebConcept npm (existing vuln DB entries).

Note: CHANGELOG entry for rulepack version 2026.05.02.2 is missing (rules were merged without a changelog entry). Separate cleanup PR needed.

Fix (same commit): removed SUP-039, MAL-076, MAL-077 from `chain_rules` — these were added in the 2026.05.02.2 update with static-rule schema (pattern/mitigation/test_input) but erroneously left in `chain_rules` as duplicates, breaking the validate.yml `duplicate rule ID` check. They remain in `static_rules` where they were correctly added.

Total: 266 static rules + 14 chain rules = 280.

Sources:
- CVE-2026-25536 (GitHub Advisory): https://github.com/modelcontextprotocol/typescript-sdk/security/advisories/GHSA-345p-7cg4-v4c7
- CVE-2026-25536 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-25536
- mcp-handler advisory (GHSA-w2fm-25vw-vh7f): https://github.com/advisories/GHSA-W2FM-25VW-VH7F

## 2026.05.02.1

Pattern update 2026-05-02. Three new PSV rules covering two critical WeKnora MCP stdio command injection vulnerabilities (CVE-2026-22688 and CVE-2026-30861) and one high-severity OS command injection in Sunwood-ai-labs/command-executor-mcp-server (CVE-2026-7593). Vuln DB additions for WeKnora and command-executor-mcp-server.

- **PSV-060** (critical, new): **WeKnora MCP stdio authenticated command injection (CVE-2026-22688, GHSA-78h3-63c4-5fqc, CVSS 9.9, CWE-77, < 0.2.5)** — Tencent WeKnora (github.com/Tencent/WeKnora), an LLM-powered document understanding framework, is vulnerable to command injection via the MCP stdio test endpoint in versions prior to 0.2.5. When `transport_type=stdio` is set, the `stdio_config.command` and `stdio_config.args` fields are passed directly to subprocess execution without sanitization. An authenticated attacker with low privileges can exploit this remotely to execute arbitrary OS commands. Patched in 0.2.5, but that patch was incomplete (see CVE-2026-30861 / PSV-061); upgrade to >= 0.2.10.
- **PSV-061** (critical, new): **WeKnora MCP stdio unauthenticated npx -p bypass RCE (CVE-2026-30861, GHSA-r55h-3rwj-hcmg, CVSS 10.0, CWE-78, 0.2.5 ≤ x < 0.2.10)** — The 0.2.5 patch for CVE-2026-22688 implemented an MCP stdio command allowlist (only `npx` and `uvx` permitted) with argument blacklists, but failed to block the `-p` flag. Attackers can execute `npx node -p "<arbitrary JS>"` to evaluate arbitrary JavaScript payloads, achieving full OS command execution. Because WeKnora permits unrestricted user registration, this vulnerability is exploitable without prior credentials, making it a zero-credential RCE on any public WeKnora instance on 0.2.5–0.2.9. Fixed in 0.2.10. Published March 7, 2026 by GHSA.
- **PSV-062** (high, new): **command-executor-mcp-server OS command injection via allowlist bypass (CVE-2026-7593, CVSS 7.3, <= 0.1.0)** — Sunwood-ai-labs/command-executor-mcp-server (a TypeScript MCP server for executing pre-approved commands) is vulnerable to OS command injection in the `execute_command` function in `src/index.ts`. The allowlist validation (permitting git, ls, mkdir, cd, npm, npx, python) can be bypassed to execute arbitrary OS commands. Disclosed publicly May 1, 2026; no patch was available at time of disclosure. Remote exploitation is possible.
- **Vuln DB additions:** `weknora` product entry: 0.0.1/0.2.0/0.2.4 → CVE-2026-22688 (critical, fixed: 0.2.5), 0.2.5/0.2.9 → CVE-2026-30861 (critical, fixed: 0.2.10). `command-executor-mcp-server` product entry: 0.1.0 → CVE-2026-7593 (high, no fix).

Total: 262 static rules + 14 chain rules = 276.

Note: CHANGELOG entry for rulepack version 2026.05.01.2 is missing (rules SUP-049 and PSV-059 were merged without a changelog entry). Separate cleanup PR needed.

Sources:
- CVE-2026-22688 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-22688
- CVE-2026-22688 (GHSA): https://github.com/Tencent/WeKnora/security/advisories/GHSA-78h3-63c4-5fqc
- CVE-2026-30861 (GHSA): https://github.com/advisories/GHSA-r55h-3rwj-hcmg
- CVE-2026-30861 (TheHackerWire): https://www.thehackerwire.com/weknora-rce-cve-2026-30861-unauthenticated-command-injection/
- CVE-2026-7593 (TheHackerWire): https://www.thehackerwire.com/vulnerability/CVE-2026-7593/
- CVE-2026-7593 (OffSeq): https://radar.offseq.com/threat/cve-2026-7593-os-command-injection-in-sunwood-ai-l-deeb59c6

## 2026.05.01.1

Fix (same-day): move MAL-075, PSV-058, ABU-009 from `chain_rules` to `static_rules`. These rules were added in recent PRs with `pattern`/`mitigation`/`test_input` fields (static-rule schema) but incorrectly placed under `chain_rules`, which requires `all_of`. The pydantic model raises `Field required: all_of` on load, breaking the bundled snapshot CI. Matches the fix pattern from 2026.04.29.2. Also corrects the chain/total count in the 2026.05.01.1 entry header from "18/272" to "14/271" (the correct post-move totals).

Pattern update 2026-05-01. One new SUP rule covering the PyTorch Lightning Mini Shai-Hulud supply chain compromise (April 30, 2026). IOC DB addition for C2 domain zero.masscan.cloud. Vuln DB addition for lightning 2.6.2 and 2.6.3.

- **SUP-048** (critical, new): **PyTorch Lightning Mini Shai-Hulud supply chain compromise (lightning 2.6.2–2.6.3, C2: zero.masscan.cloud)** — Two malicious versions of the PyTorch Lightning framework (PyPI package "lightning") — 2.6.2 and 2.6.3 — were published on 2026-04-30 and detected 18 minutes after publication by Socket. The malicious wheel bundles a hidden `_runtime` directory containing a downloader and heavily obfuscated JavaScript payload that executes automatically on module import (no user interaction required beyond installation). The payload harvests Anthropic API keys, GitHub tokens, AWS/GCP credentials, SSH keys, and environment variables, exfiltrating them encrypted to C2 domain `zero.masscan.cloud:443/v1/telemetry`. If the primary channel fails, it falls back to creating a public GitHub repository with description "A Mini Shai-Hulud has Appeared" as an out-of-band exfiltration beacon. Worm-like propagation capabilities also target GitHub and npm ecosystems using stolen tokens. Attributed to the Mini Shai-Hulud / TeamPCP threat actor. PyTorch Lightning receives millions of monthly downloads, making this a high-impact incident for AI/ML development environments. Uninstall lightning 2.6.2 and 2.6.3, upgrade to 2.6.4 or later, rotate all API keys and cloud credentials, and audit git logs for unexpected "XprobeBot" commits and GitHub repository creation.
- **IOC additions:** C2 domain `zero.masscan.cloud` (Mini Shai-Hulud lightning supply chain, TeamPCP).
- **Vuln DB additions:** `lightning` 2.6.2 and 2.6.3 (PYPI-BACKDOOR-2026-0430, critical, fixed: 2.6.4).

Total: 257 static rules + 14 chain rules = 271.

Sources:
- Socket advisory: https://socket.dev/blog/lightning-pypi-package-compromised
- The Hacker News: https://thehackernews.com/2026/04/pytorch-lightning-compromised-in-pypi.html
- Aikido Security: https://www.aikido.dev/blog/pytorch-lightning-pypi-compromise-mini-shai-hulud
- Semgrep: https://semgrep.dev/blog/2026/malicious-dependency-in-pytorch-lightning-used-for-ai-training/
- OX Security (8.3M downloads compromised): https://www.ox.security/blog/lightning-python-package-shai-hulud-supply-chain-attack/

## 2026.04.30.1

Pattern update 2026-04-30. One new PSV rule covering the GitHub Enterprise Server git push RCE disclosed April 28, 2026. Vuln DB additions for three npm CanisterSprawl packages and one GHES product entry.

- **PSV-057** (high, new): **GitHub Enterprise Server RCE via X-Stat push option injection (CVE-2026-3854, GHSA-64fw-jx9p-5j24, GHES < 3.14.25/3.15.20/3.16.16/3.17.13/3.18.8/3.19.4)** — Wiz Research disclosed on April 28, 2026 that GHES embeds user-supplied `git push --push-option` values in the internal semicolon-delimited X-Stat service header without stripping semicolons (CVSS 8.7, CWE-77). An authenticated attacker with push access can inject three header fields in a single `git push`: `rails_env` (bypass the production sandbox), `custom_hooks_dir` (redirect hook execution to an attacker-controlled directory), and `repo_pre_receive_hooks` (path traversal to execute arbitrary code as the git service user). GitHub.com was patched same-day on March 4, 2026 with no confirmed exploitation. 88% of self-hosted GHES instances remained unpatched at time of disclosure. Upgrade to 3.14.25, 3.15.20, 3.16.16, 3.17.13, 3.18.8, 3.19.4, or 3.20.0 immediately.
- **Vuln DB additions:** `@openwebconcept/design-tokens` 1.0.1–1.0.3 and `@openwebconcept/theme-owc` 1.0.1–1.0.3 (CanisterSprawl worm, same C2 as existing Namastex/SUP-047 rule: telemetry.api-monitor.com + ICP canister cjn37); `github-enterprise-server` <3.19.4 (CVE-2026-3854).

Total: 253 static rules + 14 chain rules = 267.

Sources:
- CVE-2026-3854 (Wiz Research): https://www.wiz.io/blog/github-rce-vulnerability-cve-2026-3854
- CVE-2026-3854 (GitHub Blog): https://github.blog/security/securing-the-git-push-pipeline-responding-to-a-critical-remote-code-execution-vulnerability/
- CVE-2026-3854 (Help Net Security): https://www.helpnetsecurity.com/2026/04/29/cve-2026-3854-github-rce-vulnerability/
- CVE-2026-3854 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-3854
- GHSA-64fw-jx9p-5j24: https://github.com/advisories/GHSA-64fw-jx9p-5j24
- @openwebconcept CanisterSprawl (Socket): https://socket.dev/blog/namastex-npm-packages-compromised-canisterworm

## 2026.04.29.2

Fix: move PSV-048, PSV-049, PSV-050 from `chain_rules` to `static_rules`. These rules were added in PR #18 with `pattern`/`mitigation`/`test_input` fields (static-rule schema) but incorrectly placed under `chain_rules`, which requires `all_of`. The pydantic model in `skillscan-security` raises `Field required: all_of` on load, causing the bundled snapshot to fail the cross-tool recall CI check. This matches the fix applied to MAL-074/PSV-047/PINJ-025 in commit 483d627.

Total: 248 static rules + 14 chain rules.

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
