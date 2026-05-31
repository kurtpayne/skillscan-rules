# SkillScan Rules Changelog

## 2026.05.31.1

Pattern update 2026-05-31. Two new rules: PSV-088 (OpenClaw Claw Chain sandbox escape/escalation chain) and PSV-089 (Hermes WebUI file deletion + path traversal).

- **PSV-088** (critical, new): **OpenClaw Claw Chain — sandbox escape, credential read, heredoc bypass, owner impersonation (CVE-2026-44112/44113/44115/44118, < 2026.4.22)** — Cyera Research disclosed four chainable vulnerabilities in all OpenClaw versions prior to 2026.4.22. CVE-2026-44112 (CVSS 9.6, CWE-367) and CVE-2026-44113 (CVSS 7.7, CWE-367) exploit TOCTOU race conditions in the OpenShell managed sandbox backend to escape the sandbox mount root for writes and reads respectively. CVE-2026-44115 (CVSS 8.8, CWE-184) allows shell expansion tokens embedded in heredoc bodies to bypass the OpenShell command allowlist. CVE-2026-44118 (CVSS 7.8, CWE-284) exposes an improper access control flaw where the loopback gateway trusts a client-controlled `senderIsOwner` flag without validating it against the authenticated session, allowing any loopback client to impersonate the owner and control gateway configuration, cron scheduling, and execution environment. The full attack chain — read credentials via CVE-2026-44113/44115, escalate via CVE-2026-44118, establish persistence via CVE-2026-44112 — can be executed by any authenticated agent process. Approximately 245,000 exposed instances identified via Shodan/ZoomEye (May 2026). Upgrade OpenClaw to 2026.4.22 or later.

- **PSV-089** (high, new): **Hermes WebUI arbitrary file deletion + path traversal (CVE-2026-6832/6829, hermes-webui < 0.50.34)** — CVE-2026-6832 (CVSS 8.1 HIGH, CWE-22) is an arbitrary file deletion vulnerability in the hermes-webui npm package (nesquena/hermes-webui) prior to version 0.50.34. The `/api/session/delete` endpoint constructs the deletion path by concatenating `SESSION_DIR` with the user-supplied `session_id` parameter without sanitization, allowing an authenticated attacker to delete arbitrary writable files on the host via absolute path or `../../../` traversal. CVE-2026-6829 (CVSS 5.3 MEDIUM, CWE-22) affects workspace-management endpoints (`/api/session/new`, `/api/session/update`, `/api/chat/start`, `/api/workspaces/add`) that accept user-supplied workspace path parameters without sanitization, enabling authenticated directory traversal reads. AI agent orchestration frameworks deploying Hermes as a local WebUI server (e.g., NousResearch/hermes-agent) inherit both vulnerabilities. Upgrade hermes-webui to 0.50.34 or later.

Vuln DB additions: `npm/hermes-webui` (CVE-2026-6832 CVSS 8.1 and CVE-2026-6829 CVSS 5.3, both fixed in 0.50.34, PSV-089). 2 new entries.
IOC additions: none.

Total: 311 static rules + 14 chain rules = 325.

Sources:
- Cyera Research (Claw Chain): https://www.cyera.com/blog/claw-chain-cyera-research-unveil-four-chainable-vulnerabilities-in-openclaw
- The Hacker News (Claw Chain): https://thehackernews.com/2026/05/four-openclaw-flaws-enable-data-theft.html
- CybersecurityNews (Claw Chain): https://cybersecuritynews.com/openclaw-chain-vulnerabilities/
- NVD CVE-2026-44112: https://nvd.nist.gov/vuln/detail/CVE-2026-44112
- NVD CVE-2026-6832: https://nvd.nist.gov/vuln/detail/CVE-2026-6832
- NVD CVE-2026-6829: https://nvd.nist.gov/vuln/detail/CVE-2026-6829
- Sherlock Forensics (CVE-2026-6832): https://www.sherlockforensics.com/blog/2026-04-22-cve-2026-6832.html
- SentinelOne (CVE-2026-6829): https://www.sentinelone.com/vulnerability-database/cve-2026-6829/

Candidates researched and already covered: TrapDoor npm/PyPI/crates (MAL-086), Azure Data Explorer MCP KQL injection (CVE-2026-33980, PSV rule), Splunk MCP token log disclosure (CVE-2026-20205, PSV rule), Azure MCP SSRF (CVE-2026-26118, PSV rule), nginx-ui MCP auth bypass (CVE-2026-33032, PSV rule), TanStack/Mistral Mini Shai-Hulud wave-3 (CVE-2026-45321, SUP rule), @bitwarden/cli Shai-Hulud (SUP rule), PyTorch Lightning Mini Shai-Hulud (MAL rule), Apache Doris MCP SQLi (CVE-2025-66335, PSV rule), Apache Pinot MCP auth bypass (PSV rule), Alibaba RDS MCP info disclosure (PSV rule), laravel-lang DebugElevator stealer (SUP-055), Starlette BadHost (CVE-2026-48710, PSV-087), mouse5212-super-formatter Malware-Slop (MAL-087).

## 2026.05.29.1

Pattern update 2026-05-29. One new rule: SUP-055 (laravel-lang Composer supply chain poisoning — DebugElevator stealer).

- **SUP-055** (critical, new): **laravel-lang supply chain poisoning — DebugElevator stealer via git tag rewrite (all tags backdoored 2026-05-22, C2 flipboxstudio.info)** — Four widely-used Laravel localization Composer packages were supply-chain poisoned on 2026-05-22 22:32 UTC: `laravel-lang/lang`, `laravel-lang/attributes`, `laravel-lang/http-statuses`, and `laravel-lang/actions`. An attacker with push access (via a leaked GitHub PAT) rewrote every existing git tag in all four repositories to point at malicious commits containing a `src/helpers.php` dropper wired into Composer `autoload.files` — it runs on every PHP request upon install. The dropper fetches a cross-platform DebugElevator credential stealer from `https://flipboxstudio.info/payload` (`flipboxstudio[.]info` is a typosquat of the legitimate `flipboxstudio.com`) and executes it, harvesting AWS keys, GitHub tokens, Slack tokens, Stripe secrets, SSH keys, `.env` files, JWTs, Kubernetes secrets, Vault tokens, and crypto wallet recovery phrases. Approximately 700 versions across the four packages were affected (10.3M combined installs). There is no safe tagged version — pin each dependency to a commit SHA predating 2026-05-22 22:32 UTC via upstream branch history. Rule fires on the C2 domain `flipboxstudio.info`, the `DebugElevator` binary name, or `composer require/install/update` referencing any of the four affected packages.

Vuln DB additions: `composer/laravel-lang/lang`, `composer/laravel-lang/attributes`, `composer/laravel-lang/http-statuses`, `composer/laravel-lang/actions` (all SUP-055, all tags rewritten 2026-05-22, critical). 4 new entries.
IOC additions: `flipboxstudio.info` (SUP-055 DebugElevator stealer C2). 1 new domain.

Total: 309 static rules + 14 chain rules = 323.

Sources:
- BleepingComputer: https://www.bleepingcomputer.com/news/security/laravel-lang-packages-hijacked-to-deploy-credential-stealing-malware/
- Snyk advisory: https://snyk.io/blog/laravel-lang-supply-chain-advisory/
- StepSecurity: https://www.stepsecurity.io/blog/laravel-lang-supply-chain-attack
- Aikido: https://www.aikido.dev/blog/supply-chain-attack-targets-laravel-lang-packages-with-credential-stealer
- The Hacker News: https://thehackernews.com/2026/05/laravel-lang-php-packages-compromised.html

Candidates researched and already covered: TrapDoor npm/PyPI/crates (MAL-086), CVE-2026-26118 Azure MCP SSRF (PSV rule), CVE-2026-30615 Windsurf MCP STDIO injection (PSV rule), CVE-2026-33032 nginx-ui MCP auth bypass (vuln_db), axios npm compromise (SUP rule), Nx Console v18.95.0 (MAL rule).

## 2026.05.28.1

Pattern update 2026-05-28. Two new rules: PSV-087 (Starlette BadHost auth bypass, CVE-2026-48710) and MAL-087 (Malware-Slop, mouse5212-super-formatter Claude AI data exfiltration).

- **PSV-087** (high, new): **Starlette BadHost Host-header auth bypass (CVE-2026-48710, GHSA-86qp-5c8j-p5mr, PYSEC-2026-161, < 1.0.1) — FastAPI/vLLM/LiteLLM/MCP servers affected** — CVE-2026-48710 is an authentication bypass in Starlette versions 0.8.3 through 1.0.0. Starlette reconstructs `request.url` by concatenating the raw HTTP `Host` header with the request path without validating the `Host` value against RFC 9112/RFC 3986 grammar. Because authentication middleware makes access decisions based on `request.url.path` while the ASGI routing engine uses the raw path scope, an attacker can inject an extra path component into the `Host` header to make `request.url.path` point to an unrestricted route while the actual request targets a protected endpoint. Discovered by X41 D-Sec (X41-2026-002) during an OSTIF-sponsored security audit, coordinated disclosure May 22 2026. Downstream impact is critical: every FastAPI application, vLLM inference server, LiteLLM proxy, and MCP server built on Starlette (approximately 325 million weekly downloads) inherits this auth bypass, enabling unauthenticated access to restricted LLM endpoints, credential extraction, and internal agent tool interaction. Upgrade Starlette to 1.0.1 or later. Rule fires on CVE/GHSA/PYSEC IDs or starlette + host-header auth bypass keyword combinations.

- **MAL-087** (critical, new): **Malware-Slop: mouse5212-super-formatter npm infostealer targeting Claude AI /mnt/user-data directory (May 2026)** — `mouse5212-super-formatter` is a malicious npm package (all versions) discovered May 27 2026 by OX Security. Its postinstall script masquerades as an "archive deployment sync" utility: it authenticates to GitHub using either a `GITHUB_TOKEN` environment variable found on the victim machine or a hard-coded attacker fallback token (notably leaked in the package itself — likely the result of AI-generated malware code with poor OPSEC), creates a repository under the threat actor GitHub account (`mouse5212`), and then recursively uploads every file under `/mnt/user-data` — the dedicated directory used by Anthropic Claude AI for uploads, outputs, and project files — to that remote repository. Approximately 676 downloads were recorded before removal. The leaked GitHub private token has been invalidated by GitHub. Remove the package immediately and rotate all credentials accessible in the execution environment. Audit `/mnt/user-data` for sensitive files. Rule fires on the package name, Malware-Slop campaign name + npm/claude/github context, or `/mnt/user-data` combined with GitHub exfiltration behavior.

Vuln DB additions: `starlette/1.0.0` (CVE-2026-48710, affected_range >=0.8.3 <1.0.1, PSV-087). `npm/mouse5212-super-formatter/all` (NPM-MOUSE5212-2026-MALWARE-SLOP, all versions malicious, MAL-087). 2 new entries.
IOC additions: none.

Total: 308 static rules + 14 chain rules = 322.

Sources:
- OSTIF/X41 D-Sec (BadHost): https://ostif.org/disclosing-the-badhost-vulnerability-in-starlette/
- Tenable CVE-2026-48710: https://www.tenable.com/cve/CVE-2026-48710
- The Hacker News (Malware-Slop): https://thehackernews.com/2026/05/malicious-npm-package-stole-files-from.html
- OX Security (Malware-Slop): https://www.ox.security/blog/malware-slop-new-malicious-npm-package-leaks-its-own-github-private-token/
- The Register (Malware-Slop): https://www.theregister.com/cyber-crime/2026/05/27/supply-chain-brain-drain-npm-attacker-foolishly-leaks-own-github-private-token/5247424

Candidates researched and already covered: TrapDoor npm/PyPI/crates supply chain (MAL-086, 2026-05-25), CVE-2026-45321 Mini Shai-Hulud wave-5 TanStack/Mistral (SUP rule + vuln_db), CVE-2026-23478 Cal.com JWT bypass (PSV-086), CVE-2026-33032 nginx-ui MCP auth bypass (PSV rule), CVE-2026-20205 Splunk MCP disclosure (PSV rule), CVE-2026-26118 Azure MCP SSRF (PSV rule), CVE-2025-49596 MCP Inspector RCE (PSV rule), CVE-2026-22252 LibreChat MCP STDIO RCE (PSV rule), GlassWorm OpenVSX/invisible Unicode (MAL-066 + MAL-085 family).

## 2026.05.25.1

Pattern update 2026-05-25. One new rule: MAL-086 (TrapDoor multi-ecosystem supply-chain credential stealer targeting crypto/AI-dev toolchains).

- **MAL-086** (critical, new): **TrapDoor multi-ecosystem supply-chain credential stealer targeting crypto/AI-dev toolchains (npm/PyPI/crates.io, CLAUDE.md/.cursorrules zero-width poisoning, May 2026)** — TrapDoor is a coordinated supply-chain campaign (first observed May 22 2026) deploying 34 malicious packages and 384+ artifact versions across npm (21 packages), PyPI (7 packages), and Crates.io (6 packages). Named packages include `token-usage-tracker`, `wallet-security-checker`, `defi-env-auditor`, `prompt-engineering-toolkit`, `llm-context-compressor` (npm); `cryptowallet-safety`, `defi-risk-scanner`, `eth-security-auditor` (PyPI); `sui-move-build-helper`, `move-compiler-tools` (Crates.io). The npm payload is a 1,149-line credential harvester (`trap-core.js`) executed via postinstall hook, stealing SSH keys, Sui/Solana/Aptos wallet keystores, AWS credentials, GitHub tokens, browser profile data, crypto wallet extension data, environment variables, and API keys. A distinctive feature is deliberate AI-developer targeting: the payload writes hidden zero-width Unicode prompt-injection instructions into `.cursorrules` and `CLAUDE.md` files in the project root, tricking AI coding assistants (Cursor, Claude Code) into running what appears to be a routine project security scan but is actually credential exfiltration under the guise of an automated audit. Crates.io packages use XOR encryption with hardcoded key `cargo-build-helper-2026`. Payload delivery and C2 via `ddjidd564.github.io`. Campaign marker `P-2024-001`. Attribution: GitHub account `ddjidd564`. Rule fires on any TrapDoor package name, `trap-core.js`, or the `P-2024-001` campaign marker.

Vuln DB additions: `npm/token-usage-tracker`, `npm/wallet-security-checker`, `npm/defi-env-auditor`, `npm/prompt-engineering-toolkit`, `npm/llm-context-compressor`, `pip/cryptowallet-safety`, `pip/defi-risk-scanner`, `pip/eth-security-auditor` (all MAL-086, all versions malicious). 8 new entries.
IOC additions: `ddjidd564.github.io` (MAL-086 TrapDoor C2/payload host). 1 new domain.

Total: 306 static rules + 14 chain rules = 320.

Sources:
- Socket.dev (primary): https://socket.dev/blog/trapdoor-crypto-stealer-npm-pypi-crates
- The Hacker News: https://thehackernews.com/2026/05/trapdoor-supply-chain-attack-spreads.html
- GBHackers: https://gbhackers.com/hackers-compromise-34-npm-pypi-and-crates-packages/
- The Block: https://www.theblock.co/post/402458/researchers-flag-trapdoor-malware-campaign-targeting-crypto-developer-environments-including-aptos-sui-and-solana

Candidates researched and already covered: Zero-width Unicode steganography (EVASION-004), Clinejection prompt injection (PINJ-005), Cline@2.3.0 openclaw supply chain (vuln_db NPM-CLINE-2026-03), Mini Shai-Hulud TanStack/Mistral wave-3 (MAL-081), Mini Shai-Hulud @antv wave-4 (MAL-084), Mini Shai-Hulud Nx Console wave-4 (MAL-085), CVE-2026-26118 Microsoft MCP SSRF (PSV rule), CVE-2026-20205 Splunk MCP disclosure (PSV rule), CVE-2026-23478 Cal.com JWT bypass (PSV-086), Cline kanban WebSocket CVE-2026-44211 (PSV-074), GlassWorm OpenVSX extension (MAL-066).

## 2026.05.24.1

Pattern update 2026-05-24. One new rule: PSV-086 (Cal.com JWT callback authentication bypass, CVE-2026-23478, CVSS 10.0).

- **PSV-086** (critical, new): **Cal.com JWT callback authentication bypass — full account takeover (CVE-2026-23478, GHSA-7hg4-x4pr-3hrg, CVSS 10.0 Critical, CWE-602/CWE-639, 3.1.6–6.0.6)** — Cal.com versions 3.1.6 through 6.0.6 contain a critical authentication bypass in the custom NextAuth JWT callback (`packages/features/auth/lib/next-auth-options.ts`). When the callback is triggered with `trigger === "update"`, it accepts identity fields — including `email` — supplied by the client via `session.update()` without server-side validation. Because Cal.com uses `token.email` as the primary database lookup key to reconstruct the session, any attacker can call `session.update({ email: "victim@example.com" })` to acquire a valid JWT for an arbitrary account, enabling full account takeover with no elevated privileges required. AI scheduling agents that connect to a self-hosted Cal.com instance running 3.1.6–6.0.6 are directly exposed: a prompt injection attack or a compromised MCP client could trigger this bypass to exfiltrate meeting data or impersonate any user. Upgrade Cal.com to 6.0.7 and force session invalidation for all existing sessions. Rule fires on CVE/GHSA ID or cal.com + session.update/jwt callback + account takeover keyword combinations.

Vuln DB additions: `calcom/6.0.6` representative of >=3.1.6 <6.0.7 (CVE-2026-23478, PSV-086). 1 new entry.
IOC additions: none.

Total: 305 static rules + 14 chain rules = 319.

Sources:
- GHSA-7hg4-x4pr-3hrg (Cal.com): https://github.com/calcom/cal.com/security/advisories/GHSA-7hg4-x4pr-3hrg
- CyberPress: https://cyberpress.org/critical-cal-com-flaw-2/
- CVE NVD: https://nvd.nist.gov/vuln/detail/CVE-2026-23478

Candidates researched and already covered: Mini Shai-Hulud wave-4 @antv ecosystem (MAL-084), Nx Console v18.95.0 compromise (MAL-085), node-ipc domain takeover supply chain (MAL-083), CVE-2026-44338 PraisonAI missing auth (PSV-085), GHSA-wpqr-6v78-jr5g Gemini CLI CI RCE (PSV-083), CVE-2026-30615 Windsurf zero-click MCP injection (PSV-082 family), CVE-2026-23478 Cal.com JWT bypass (PSV-086 — new today), CVE-2026-33032 nginx-ui MCP auth bypass (vuln_db), CVE-2026-20205 Splunk MCP disclosure (vuln_db), CVE-2026-33980 Azure Data Explorer MCP SQLi (vuln_db), CVE-2025-66335 Apache Doris MCP SQLi (vuln_db), CVE-2026-39974 n8n-MCP SSRF (PSV rule), Context.ai/Vercel breach (PSV rule).

## 2026.05.23.1

Pattern update 2026-05-23. Two new rules: MAL-085 (Nx Console v18.95.0 VS Code extension supply chain compromise) and SUP-054 (durabletask v1.4.1–1.4.3 PyPI supply chain compromise).

- **MAL-085** (critical, new): **Mini Shai-Hulud wave-4 (TeamPCP) Nx Console VS Code extension supply chain compromise (nrwl.angular-console v18.95.0, May 18 2026)** — On May 18 2026 (12:30–12:48 UTC) the nrwl.angular-console VS Code extension (Nx Console, 2.2M installs) was trojanized and published to the Visual Studio Marketplace for 18 minutes. The compromised v18.95.0 silently fetched and executed a 498 KB obfuscated payload hidden in a dangling orphan commit in the official nrwl/nx GitHub repository. The payload harvests GitHub tokens, npm access tokens, AWS credentials (env + IMDS + Secrets Manager), HashiCorp Vault tokens, Kubernetes service account tokens, 1Password vault data, and ~/.claude/settings.json. Data is exfiltrated via three channels: HTTPS, GitHub API dead-drops, and DNS tunneling. A Python backdoor (cat.py) provides persistent access using the GitHub Search API as a command dead-drop signed with an attacker RSA-4096 key. Approximately 3,800 GitHub internal repositories were exfiltrated (including ~3,800 from GitHub's own estate). The root cause was a developer's machine infected by @tanstack/zod-adapter@1.166.15 (TeamPCP wave-3, May 11), seven days prior — demonstrating worm-to-extension supply chain propagation. Attribution: TeamPCP. Rule fires on the extension ID + version or campaign+product keyword combinations.

- **SUP-054** (critical, new): **durabletask v1.4.1–1.4.3 PyPI supply chain compromise (TeamPCP wave-4, C2: check.git-service.com, Linux wiper + cloud credential theft, May 19 2026)** — Three malicious versions of Microsoft's official Python Durable Task client (durabletask 1.4.1, 1.4.2, 1.4.3) were published by TeamPCP on May 19 2026 and subsequently quarantined. The dropper is injected directly into Python source files; on import it fetches rope.pyz from https://check.git-service.com/rope.pyz (domain registered May 16 2026) and executes the second-stage payload, which deploys a Linux disk wiper, harvests cloud credentials from every major provider, and exfiltrates them via an attacker RSA-encrypted channel. Part of TeamPCP wave-4 (same day as the @antv npm ecosystem compromise, MAL-084). Rule fires on the C2 domain, payload filename, compromised version references, or pip install of any affected version.

Vuln DB additions: `pip/durabletask` 1.4.1, 1.4.2, 1.4.3 (SUP-054, PYPI-DURABLETASK-2026-TEAMPCP-WAVE4). 3 new entries.
IOC additions: `check.git-service.com` (SUP-054 C2 domain). 1 new domain.

Total: 304 static rules + 14 chain rules = 318.

Sources:
- The Hacker News (Nx Console): https://thehackernews.com/2026/05/compromised-nx-console-18950-targeted.html
- Nx Blog postmortem: https://nx.dev/blog/nx-console-v18-95-0-postmortem
- StepSecurity (Nx Console): https://www.stepsecurity.io/blog/nx-console-vs-code-extension-compromised
- Aikido (durabletask): https://www.aikido.dev/blog/durabletask-package-compromised-mini-shai-hulud
- CyberPress (durabletask): https://cyberpress.org/microsoft-durabletask-python-client-compromised/
- Phoenix Security (wave-4): https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/

Candidates researched and already covered: CVE-2026-26118 Azure MCP SSRF (PSV rule), CVE-2026-20205 Splunk MCP token disclosure (PSV rule), CVE-2026-33980 Azure Data Explorer MCP KQL injection (PSV-043), Mini Shai-Hulud waves 1–3 (SUP-039, SUP-042, SUP-048–050, MAL-078–081), Mini Shai-Hulud wave-4 @antv (MAL-084), Cline kanban WebSocket hijacking CVE-2026-44211 (PSV-074), GlassWorm extension campaign (MAL-066 + multiple GlassWorm rules), Clinejection prompt injection (PINJ-005).

## 2026.05.21.1

Pattern update 2026-05-21. Four new rules: MAL-084 (@antv npm wave-4), PSV-083 (Gemini CLI CI RCE), PSV-084 (Cursor git hook sandbox escape), PSV-085 (PraisonAI missing auth).

- **MAL-084** (critical, new): **Mini Shai-Hulud wave-4 (TeamPCP) @antv npm ecosystem compromise via maintainer account takeover (echarts-for-react@3.2.7, C2: t.m-kosche.com, May 19 2026)** — On May 19 2026 (01:56–02:18 UTC) the npm maintainer account `atool` was compromised and 637 malicious versions were published across 317 packages in the `@antv` ecosystem and adjacent packages (`size-sensor`, `echarts-for-react`, `timeago.js`, `jest-canvas-mock`, `jest-date-mock`) in a 22-minute automated burst. The payload is a 498 KB obfuscated Bun bundle (`index.js`) delivered as a preinstall hook. It harvests GitHub PATs, AWS credentials (env, config, EC2 IMDS, ECS metadata, Secrets Manager), Kubernetes service account tokens, HashiCorp Vault tokens, npm access tokens, SSH keys, and 1Password/Bitwarden/pass vault data, exfiltrating to `t.m-kosche.com:443` and the Session P2P network (`filev2.getsession.org`). A Python backdoor (`cat.py`) provides persistent remote access. GitHub removed 640 malicious packages and invalidated 61,274 npm granular access tokens. Any project with `"echarts-for-react": "^3.0.6"` resolved to `3.2.7` (malicious) on a clean install. Attribution: TeamPCP (same operators as waves 1–3, same Bun obfuscation toolkit). Rule fires on the new C2 domain `t.m-kosche.com`, the compromised version `echarts-for-react@3.2.7`, or campaign+scope references.

- **PSV-083** (critical, new): **Google Gemini CLI CI headless folder-trust bypass + --yolo tool-allowlist bypass RCE (GHSA-wpqr-6v78-jr5g, @google/gemini-cli < 0.39.1, CVSS 10.0, CWE-20/77/78)** — GHSA-wpqr-6v78-jr5g is a dual RCE in `@google/gemini-cli` prior to 0.39.1 (and `= 0.40.0-preview.2`). (1) **Folder Trust Bypass**: headless/CI mode automatically trusted workspace folders for loading `.env` and `.gemini/settings.json` without user consent, enabling RCE from a malicious config file in any untrusted repository clone. (2) **--yolo Tool Allowlist Bypass**: fine-grained tool restrictions were silently ignored in `--yolo` mode, enabling prompt-injection-triggered RCE with shell command permissions. No CVE assigned. Upgrade to 0.39.1 or 0.40.0-preview.3. CVSS 10.0 (AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H). Rule fires on GHSA ID, `@google/gemini-cli` with vulnerable version reference, or headless CI / folder trust / yolo allowlist bypass keywords.

- **PSV-084** (high, new): **Cursor IDE sandbox escape via .git hook write (CVE-2026-26268, GHSA-8pcm-8jpx-hv8r, CVSS 8.1, CWE-862, Cursor < 2.5)** — The Cursor IDE AI agent sandbox failed to restrict write access to `.git` configuration files. Prompt injection via a malicious repository can instruct the AI agent to write an executable payload to `.git/hooks/pre-commit` (or any other hook). The payload executes outside the sandbox with full user filesystem permissions the next time the victim runs a git operation that triggers the hook. Fixed in Cursor 2.5 with proper authorization controls on sandbox writes to `.git/` files. Rule fires on CVE/GHSA ID or Cursor + git hook + sandbox escape keyword patterns.

- **PSV-085** (high, new): **PraisonAI legacy API server missing authentication (CVE-2026-44338, GHSA-6rmh-7xcm-cpxj, CVSS 7.3, CWE-306, 2.5.6–4.6.33)** — The legacy Flask API server in PraisonAI hard-codes `AUTH_ENABLED = False` and `AUTH_TOKEN = None`. The authentication check always returns `True` when authentication is disabled, so both `/agents` (exposes agent configuration) and `/chat` (triggers workflow execution) fail open to any unauthenticated caller. Deployment templates recommend binding to `0.0.0.0`, making this externally exploitable. Active scanning began within 3h44m of the May 14 2026 advisory publication. Upgrade to praisonai 4.6.34. Rule fires on CVE/GHSA ID, `praisonai` + `AUTH_ENABLED = False`, or unauthenticated `/agents`/`/chat` endpoint references.

Vuln DB additions: `echarts-for-react@3.2.7` (MAL-084 supply chain), `@google/gemini-cli <0.39.1` + `=0.40.0-preview.2` (PSV-083), `pip/praisonai >=2.5.6,<4.6.34` (PSV-085), `cursor/<2.5` (PSV-084). 5 new entries.
IOC additions: `t.m-kosche.com` (MAL-084 C2 domain). 1 new domain.

Total: 302 static rules + 14 chain rules = 316.

Sources:
- Microsoft Security Blog (@antv wave-4): https://www.microsoft.com/en-us/security/blog/2026/05/20/mini-shai-hulud-compromised-antv-npm-packages-enable-ci-cd-credential-theft/
- The Hacker News (@antv): https://thehackernews.com/2026/05/mini-shai-hulud-pushes-malicious-antv.html
- Socket (@antv): https://socket.dev/blog/antv-packages-compromised
- GHSA-wpqr-6v78-jr5g (Gemini CLI): https://github.com/advisories/GHSA-wpqr-6v78-jr5g
- CSA Research Note (Gemini CLI CVSS 10): https://labs.cloudsecurityalliance.org/research/csa-research-note-gemini-cli-rce-cvss10-ai-tool-security-202/
- GHSA-6rmh-7xcm-cpxj (PraisonAI): https://github.com/advisories/GHSA-6rmh-7xcm-cpxj
- Sysdig (PraisonAI exploitation): https://www.sysdig.com/blog/cve-2026-44338-praisonai-authentication-bypass-in-under-4-hours-and-the-growing-trend-of-rapid-exploitation
- Sentinelone (CVE-2026-26268 Cursor): https://www.sentinelone.com/vulnerability-database/cve-2026-26268/

Candidates researched and already covered: CVE-2026-26118 Azure MCP SSRF (PSV in existing rules), Mini Shai-Hulud waves 1–3 (MAL-081, vuln_db), CVE-2026-33032 nginx-ui (vuln_db), CVE-2026-20205 Splunk MCP (vuln_db), CVE-2026-25536 MCP cross-client leak (vuln_db), CVE-2025-65717 Live Server exfiltration (vuln_db), CVE-2026-22252 LibreChat (PSV rule), CVE-2026-22688 WeKnora (PSV rule), CVE-2026-45321 TanStack/Mistral (MAL-081), Apache Doris MCP SQL injection (PSV rule), Alibaba RDS MCP (PSV-079), Pinot MCP (PSV-080).

## 2026.05.20.1

Pattern update 2026-05-20. Two new rules: MAL-083 (PromptMink DPRK crypto credential stealer) and SUP-053 (npm typosquatting Microsoft AI packages with Claude Code SessionStart hook backdoor).

- **MAL-083** (critical, new): **PromptMink DPRK (Famous Chollima) crypto credential stealer via trojanized npm/PyPI packages (@validate-sdk/v2, @solana-launchpad/sdk, scraper-npm, C2: ipfs-url-validator.vercel.app)** — PromptMink is a multi-stage credential-stealing campaign attributed to Famous Chollima (North Korea), disclosed by ReversingLabs on April 29, 2026. The primary malicious npm payload, @validate-sdk/v2, masquerades as a data-validation library. It targets crypto wallet configuration, SSH keys, and cloud credentials, exfiltrating to `ipfs-url-validator.vercel.app/fetchbs58` via a Base64-encoded C2 URL. The first-stage bait, @solana-launchpad/sdk, contains no malicious code itself but silently pulls in @validate-sdk/v2 as a transitive dependency. The campaign gained notoriety when it entered the open-source autonomous crypto trading project `openpaw-graveyard` via a commit co-authored by Claude Opus (February 28, 2026). A PyPI port, `scraper-npm`, reimplements the same stealer in Python and adds SSH-key persistence. ReversingLabs tracked 60+ packages and 300+ versions over a seven-month period (October 2025–April 2026). Rule fires on the primary payload name, PyPI port, or C2 domain.

- **SUP-053** (critical, new): **npm typosquatting Microsoft AI packages with Claude Code SessionStart hook backdoor (microsoft-applicationinsights-common v3.4.2 / ms-graph-types v2.43.2, C2: 207.90.194.2:443)** — Five typosquatting npm packages published by accounts named `superbase` and `micresoft` ship identical 4.5 MB UPX-compressed ELF binaries inside a `.claude/` directory. Two confirmed packages: `microsoft-applicationinsights-common` v3.4.2 (typosquat of `@microsoft/applicationinsights-common`) and `ms-graph-types` v2.43.2 (typosquat of `@types/microsoft-graph`), both mirroring the legitimate packages' current version numbers to evade semver-range audits. The binary executes on `npm install` and, by hijacking the Claude Code `SessionStart` hook, re-executes on every subsequent Claude Code session start in the affected project — meaning a single install creates persistent re-execution until the package is removed. The binary exfiltrates environment variables, home directory contents, git credentials, and `/proc/` entries to `207.90.194.2:443`. VirusTotal: `Program:Script/Wacapew.A!ml`. Discovered by SafeDep, May 13, 2026. Use `@microsoft/applicationinsights-common` and `@types/microsoft-graph` (scoped) exclusively.

Vuln DB additions: `@validate-sdk/v2` (npm, MAL-083, all versions), `@solana-launchpad/sdk` (npm, MAL-083, bait), `scraper-npm` (pip, MAL-083, all versions), `microsoft-applicationinsights-common` (npm, SUP-053, v3.4.2), `ms-graph-types` (npm, SUP-053, v2.43.2). 5 new entries.
IOC additions: `207.90.194.2` (SUP-053 C2 IP), `ipfs-url-validator.vercel.app` (MAL-083 C2 domain). 1 new IP + 1 new domain.

Total: 298 static rules + 14 chain rules = 312.

Sources:
- ReversingLabs: https://www.reversinglabs.com/blog/claude-promptmink-malware-crypto
- The Hacker News (DPRK wave): https://thehackernews.com/2026/04/new-wave-of-dprk-attacks-uses-ai.html
- CyberSecurityNews (PromptMink): https://cybersecuritynews.com/claude-generated-commit-adds-promptmink-malware/
- SafeDep (Claude Code hooks): https://safedep.io/malicious-npm-packages-claude-code-hooks/

Candidates researched and already covered or excluded: CVE-2026-26118 Azure MCP SSRF (PSV covered in prior runs), Cline@2.3.0 OpenClaw (already in vuln_db NPM-CLINE-2026-03), Axios supply chain (already in vuln_db + IOC sfrclak.com), Mini Shai-Hulud wave (MAL-081 covers by C2), SmartLoader/StealC (MAL-082), nginx-ui CVE-2026-33032 (PSV covered), GlassWorm/OpenVSX (MAL-066), Claude Code SessionStart hook general pattern (existing rule at line 4144).

## 2026.05.19.1

Pattern update 2026-05-19. One new rule: MAL-082 (SmartLoader/StealC LuaJIT loader distributed via fake MCP repos, Polygon blockchain dead-drop C2).

- **MAL-082** (high, new): **SmartLoader/StealC MCP-fake-repo campaign — Polygon blockchain dead-drop C2 (polygon.drpc.org eth\_call) + LuaJIT lua51.dll loader** — SmartLoader is a LuaJIT-based infostealer loader distributed via 109+ fake GitHub repositories that clone legitimate MCP server projects (including Oura Ring MCP). Each malicious ZIP contains a one-line batch launcher, LuaJIT 2.1, lua51.dll, and an obfuscated Lua payload disguised as .txt/.log. SmartLoader queries a Polygon smart contract via JSON-RPC eth\_call to polygon.drpc.org (dead-drop resolver) for the current C2 URL, then downloads and reflectively loads a StealC PE. Persistence via two scheduled tasks under %LOCALAPPDATA% masquerading as audio manager or Office components. Active as of April 2026; 109 repos across 103 GitHub accounts confirmed. Rule fires on polygon.drpc.org + eth\_call/JSON-RPC context, or lua51.dll + batch/log/txt context.

Vuln DB additions: none.
IOC additions: none (polygon.drpc.org is a legitimate service being abused; no dedicated C2 domain identified in open-source reporting).

Total: 296 static rules + 14 chain rules = 310.

Sources:
- The Hacker News: https://thehackernews.com/2026/02/smartloader-attack-uses-trojanized-oura.html
- GBHackers: https://gbhackers.com/109-fake-github-repos/
- Hexastrike: https://hexastrike.com/resources/blog/threat-intelligence/cloned-loaded-and-stolen-how-109-fake-github-repositories-delivered-smartloader-and-stealc/
- SOCPrime: https://socprime.com/active-threats/smartloader-analysis/

Candidates researched and already covered or excluded: Mini Shai-Hulud wave-3 UiPath/@opensearch-project (MAL-081 covers by C2 domain, consistent with prior exclusion note; no primary-source version list verifiable), CVE-2026-30615 Windsurf (PSV already covered), CVE-2026-26118 Azure MCP SSRF (PSV already covered), CVE-2026-22252 LibreChat (PSV already covered), CVE-2026-22688 WeKnora (PSV already covered), CVE-2025-49596 MCP Inspector (PSV already covered), CVE-2025-54136 Cursor MCPoison (PSV already covered), CVE-2025-54994 @akoskm/create-mcp-server-stdio (PSV already covered), CVE-2026-44284 FastGPT (PSV already covered), CVE-2026-45321 TanStack/Mistral AI wave-3 (MAL-081 already covered), node-ipc (SUP-052 already covered).

## 2026.05.17.1

Pattern update 2026-05-17. Two new rules: PSV-081 (Semantic Kernel Python InMemoryVectorStore eval injection RCE, CVE-2026-26030), PSV-082 (Semantic Kernel .NET SessionsPythonPlugin arbitrary file write, CVE-2026-25592).

- **PSV-081** (high, new): **Semantic Kernel Python InMemoryVectorStore eval injection RCE (CVE-2026-26030, < 1.39.4)** — CVE-2026-26030 (CWE-95) is a code injection vulnerability in the Microsoft Semantic Kernel Python library prior to 1.39.4. The InMemoryVectorStore default filter function uses unsafe string interpolation with `eval()` on parameters derived from AI model output. An attacker who can influence model responses (via prompt injection) can inject arbitrary Python code and achieve remote code execution on the agent host. Fixed in semantic-kernel 1.39.4. Discovered and disclosed by Microsoft Security Response Center, May 2026.

- **PSV-082** (high, new): **Semantic Kernel .NET SessionsPythonPlugin arbitrary file write (CVE-2026-25592, < 1.71.0)** — CVE-2026-25592 (CWE-434/CWE-73) is an arbitrary file write vulnerability in the Microsoft Semantic Kernel .NET SDK prior to 1.71.0. The `DownloadFileAsync` function in `SessionsPythonPlugin` was inadvertently decorated with `[KernelFunction]`, exposing the `localFilePath` parameter to AI model control with no path validation or directory restriction. Attackers can chain `ExecuteCode` and `DownloadFileAsync` to write files to arbitrary host paths (e.g. startup folders), escaping container sandbox isolation and achieving persistent RCE. Fixed in Microsoft.SemanticKernel 1.71.0. Discovered and disclosed by Microsoft Security Response Center, May 2026.

Vuln DB additions: `semantic-kernel` (Python) 1.39.3 (id: CVE-2026-26030, high, fixed: 1.39.4); `microsoft.semantickernel` (.NET) 1.70.0 (id: CVE-2026-25592, high, fixed: 1.71.0). 2 new entries.
IOC additions: none.

Total: 295 static rules + 14 chain rules = 309.

Sources:
- Microsoft Security Blog (May 7, 2026): https://www.microsoft.com/en-us/security/blog/2026/05/07/prompts-become-shells-rce-vulnerabilities-ai-agent-frameworks/
- NVD CVE-2026-26030: https://nvd.nist.gov/vuln/detail/CVE-2026-26030
- NVD CVE-2026-25592: https://nvd.nist.gov/vuln/detail/CVE-2026-25592
- Semantic Kernel GitHub: https://github.com/microsoft/semantic-kernel

Candidates researched and already covered or excluded: CVE-2026-5058 aws-mcp-server (PSV covered), CVE-2026-20205 Splunk MCP (PSV covered), CVE-2026-26118 Azure MCP SSRF (PSV covered), CVE-2025-66335 Apache Doris MCP (PSV-078), PSV-079 Alibaba RDS MCP (covered), PSV-080 Apache Pinot MCP (covered), TeamPCP/Mini Shai-Hulud all waves (MAL/SUP rules), @mistralai/@tanstack wave-3 (MAL-081), node-ipc compromise (SUP-052).

## 2026.05.16.1

Pattern update 2026-05-16. Three new rules: PSV-078 (Apache Doris MCP Server SQLi, CVE-2025-66335), PSV-079 (Alibaba Cloud RDS OpenAPI MCP Server unauthenticated RAG info disclosure, no CVE), PSV-080 (Apache Pinot/StarTree MCP Server SQLi via auth bypass, no CVE).

- **PSV-078** (medium, new): **Apache Doris MCP Server SQL injection (CVE-2025-66335, GHSA-qhfq-gvvc-5q6q, doris-mcp-server < 0.6.1)** — CVE-2025-66335 (CVSS 5.3, CWE-89) is a SQL injection vulnerability in the Apache Doris MCP Server (PyPI: `doris-mcp-server`) affecting versions >= 0.1.0 and < 0.6.1. The MCP query execution interface fails to adequately validate and neutralize input parameters before constructing SQL statements, allowing attackers to inject additional SQL syntax that alters query behavior, executes unintended statements, and bypasses intended access restrictions. Fixed in `doris-mcp-server` 0.6.1. Discovered by Tomer Peled (Akamai), reported May 2026.

- **PSV-079** (medium, new): **Alibaba Cloud RDS OpenAPI MCP Server unauthenticated RAG tool info disclosure (alibabacloud-rds-openapi-mcp-server, all versions, no fix)** — The Alibaba Cloud RDS OpenAPI MCP Server does not authenticate users before invoking the RAG (retrieval-augmented generation) MCP tool. Any client that can reach the MCP endpoint can invoke the RAG tool without credentials, potentially accessing sensitive metadata from the vector index including table names, schema definitions, and database structure. Alibaba declined to patch this vulnerability (classified as "not applicable"). All published versions affected. No fix available. Reported to CERT/CC by Tomer Peled (Akamai), May 2026.

- **PSV-080** (high, new): **Apache Pinot (StarTree) MCP Server SQL injection via auth bypass (mcp-pinot-server, <= 1.1.0)** — The Apache Pinot MCP Server (PyPI: `mcp-pinot-server`, maintained by StarTree at github.com/startreedata/mcp-pinot) versions through 1.1.0 have an authentication bypass that enables SQL injection against the connected Pinot instance. The MCP endpoint does not enforce authentication by default, allowing unauthenticated callers to execute arbitrary Pinot SQL queries. In externally-exposed environments this enables full database takeover. StarTree has added an OAuth option for HTTP transport but SQL injection persists in the code. No CVE assigned as of May 2026. Reported by Tomer Peled (Akamai).

Vuln DB additions: `doris-mcp-server` 0.1.0–0.5.1 (id: CVE-2025-66335, medium, fixed: 0.6.1); `alibabacloud-rds-openapi-mcp-server` 0.1.0/0.2.0/1.0.0 (id: PSV-079, medium, no fix); `mcp-pinot-server` 0.1.0/1.0.0/1.1.0 (id: PSV-080, high, no fix). 12 new entries.
IOC additions: none.

Total: 293 static rules + 14 chain rules = 307.

Sources:
- The Register (Tomer Peled/Akamai report): https://www.theregister.com/security/2026/05/13/bug-hunter-tracks-down-three-serious-mcp-database-flaws-one-left-unpatched/
- GitHub Advisory (CVE-2025-66335): https://github.com/advisories/GHSA-qhfq-gvvc-5q6q
- NVD (CVE-2025-66335): https://nvd.nist.gov/vuln/detail/CVE-2025-66335
- Apache Doris MCP Server: https://github.com/apache/doris-mcp-server
- Alibaba Cloud RDS MCP: https://github.com/aliyun/alibabacloud-rds-openapi-mcp-server
- StarTree Pinot MCP: https://github.com/startreedata/mcp-pinot

Candidates researched and already covered or excluded: CVE-2026-5058 aws-mcp-server (PSV covered), CVE-2026-33980 Azure Data Explorer MCP (PSV covered), CVE-2026-20205 Splunk MCP (PSV covered), CVE-2026-26118 Azure MCP SSRF (PSV covered), CVE-2025-65717 VS Code Live Server (PSV covered), CVE-2026-33032 nginx-ui MCP (PSV covered), TeamPCP/Shai-Hulud all waves (multiple SUP/MAL rules), MAL-081 @mistralai/@tanstack wave-3 (covered), SUP-052 node-ipc (covered yesterday).

## 2026.05.15.1

Pattern update 2026-05-15. One new rule: SUP-052 (node-ipc supply chain via expired maintainer email domain, May 14, 2026).

- **SUP-052** (critical, new): **node-ipc supply chain compromise via expired maintainer email domain takeover (9.1.6/9.2.3/12.0.1, C2: sh.azurestaticprovider.net)** — On May 14, 2026, versions 9.1.6, 9.2.3, and 12.0.1 of the node-ipc npm package (3.35M+ monthly downloads) were published through a compromised maintainer account. The attacker re-registered the dormant maintainer's expired email domain (atlantis-software.net) via Namecheap, triggering a standard npm password-reset flow to gain publish rights — no vulnerability in npm required. The 80 KB obfuscated payload silently harvests 90+ credential categories (AWS, GCP, Azure, GitHub CLI, Kubernetes, HashiCorp Vault, Claude AI settings, SSH keys, shell history, database passwords), compresses them into a gzip archive, and exfiltrates to sh.azurestaticprovider.net masquerading as Azure infrastructure. A secondary DNS-TXT channel overrides the system resolver to 8.8.8.8 to bypass local DNS inspection. The malware does not establish persistence. This is a separate campaign from Mini Shai-Hulud (TeamPCP) — no OIDC token hijacking, no self-propagating worm. Rule fires on the C2 domain, the expired maintainer domain, and node-ipc pinned to any of the three compromised version strings.

Vuln DB additions: `node-ipc` 9.1.6, 9.2.3, 12.0.1 (id: SUP-052, critical, fixed: 9.2.1 / 12.0.0). 3 new entries.
IOC additions: `sh.azurestaticprovider.net` (C2), `atlantis-software.net` (expired maintainer domain). 2 new domains.

Total: 290 static rules + 14 chain rules = 304.

Sources:
- The Hacker News: https://thehackernews.com/2026/05/stealer-backdoor-found-in-3-node-ipc.html
- StepSecurity advisory: https://www.stepsecurity.io/blog/node-ipc-npm-supply-chain-attack
- SafeDep analysis: https://safedep.io/malicious-node-ipc-npm-compromise/
- Socket: https://socket.dev/blog/node-ipc-package-compromised

Candidates researched and already covered or excluded: nginx-ui CVE-2026-33032 (PSV covered in prior run), Splunk MCP CVE-2026-20205 (PSV covered in prior run), CVE-2025-65717 Live Server (PSV covered in prior run), Mini Shai-Hulud wave-3 UiPath/@opensearch-project (MAL-081 covers by C2 domain; specific versions excluded — no primary-source version list verifiable, consistent with 2026.05.14.1 exclusion), nginx-ui CVE-2026-33032 (already covered).

## 2026.05.14.1

Pattern update 2026-05-14. No new rules. Vuln DB enrichment only: version-specific entries for @tanstack/react-router and @tanstack/router-core compromised in Mini Shai-Hulud wave-3 (CVE-2026-45321).

Vuln DB additions: `@tanstack/react-router` 1.169.5 (id: MAL-081, critical, CVE-2026-45321, fixed: 1.169.6), `@tanstack/router-core` 1.169.5 (id: MAL-081, critical, CVE-2026-45321, fixed: 1.169.6). 2 new entries.
IOC additions: none.

Total: 289 static rules + 14 chain rules = 303 (unchanged).

Sources:
- GitHub mini-shai-hulud-checker: https://github.com/champjss/mini-shai-hulud-checker-20260512
- TanStack GitHub issue #7383: https://github.com/TanStack/router/issues/7383
- CVE-2026-45321 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-45321

Candidates researched and already covered or excluded: CVE-2026-5029 code-runner-mcp (PSV-077, covered 2026.05.13.1), CVE-2026-43901 wireshark-mcp (PSV-076, covered 2026.05.13.1), mistralai 2.4.6 PyPI (vuln_db covered 2026.05.13.1), guardrails-ai 0.10.1 PyPI (vuln_db covered 2026.05.13.1), @opensearch-project/opensearch compromise (insufficient primary source confirmation — version list came from aggregated reports, no verifiable primary source available), TanStack MAL-081 rule (covered 2026.05.12.1), nginx-ui CVE-2026-33032 (covered), @yoda.digital/gitlab-mcp-server PSV-072 (covered), @profullstack/mcp-server PSV-073 (covered).

## 2026.05.13.1

Pattern update 2026-05-13. Two new rules: PSV-076 (wireshark-mcp path traversal, CVE-2026-43901), PSV-077 (code-runner-mcp unauthenticated RCE, CVE-2026-5029). Plus PyPI vuln DB enrichment for Mini Shai-Hulud wave-3.

- **PSV-076** (medium, new): **wireshark-mcp arbitrary directory write via unvalidated dest_dir in wireshark_export_objects (CVE-2026-43901, <= 1.1.5, CVSS 6.8)** — Disclosed May 11, 2026. The `wireshark_export_objects` MCP tool in wireshark-mcp 1.1.5 and earlier accepts an attacker-controlled `dest_dir` parameter and passes it to tshark's `--export-objects` flag without path restriction. The `_allowed_dirs` sandbox defaults to `None` and only activates when `WIRESHARK_MCP_ALLOWED_DIRS` is explicitly set, so any filesystem path is a valid write target in a default deployment. An MCP client can overwrite sensitive files, plant backdoors, or escalate privileges on misconfigured systems. No fix available; mitigate via `WIRESHARK_MCP_ALLOWED_DIRS` and network-level access control. Rule fires on CVE ID, `wireshark-mcp` + `wireshark_export_objects`/`dest_dir`/path-traversal/WIRESHARK_MCP_ALLOWED_DIRS context.

- **PSV-077** (high, new): **@mcpc-tech/code-runner-mcp unauthenticated HTTP /mcp endpoint allows RCE (CVE-2026-5029, all versions, no fix)** — Reported by CERT Polska, May 2026. When `@mcpc-tech/code-runner-mcp` is started with `--transport http`, it exposes a `/mcp` JSON-RPC endpoint on port 3088 without any authentication. Any network caller can invoke code-execution tools and run arbitrary code with server process privileges. No fix available; mitigate by binding to loopback only (`--host 127.0.0.1`) and enforcing network-level access control. Rule fires on CVE ID, `code-runner-mcp` + `--transport http`/port-3088/unauthenticated/arbitrary-code context, and `mcpc-tech` + same keywords.

Vuln DB additions: `pip/mistralai` 2.4.6 (CVE-2026-45321 / Mini Shai-Hulud wave-3, REMOVED), `pip/guardrails-ai` 0.10.1 (same), `wireshark-mcp` <= 1.1.5 (CVE-2026-43901, medium), `@mcpc-tech/code-runner-mcp` all (CVE-2026-5029, high). 4 new entries.
IOC additions: none (git-tanstack.com already in DB from 2026.05.12.1; Session P2P infrastructure excluded to avoid FPs).

Total: 289 static rules + 14 chain rules = 303.

Sources:
- CVE-2026-43901 (CIRCL): https://vulnerability.circl.lu/vuln/cve-2026-43901
- CVE-2026-43901 (ThreatInt): https://cve.threatint.eu/CVE/CVE-2026-43901
- CVE-2026-5029 (CERT Polska): https://cert.pl/en/posts/2026/05/CVE-2026-5029/
- Code Runner MCP repository: https://github.com/mcpc-tech/code-runner-mcp
- mistralai PyPI attack: https://dev.to/cryip/mistral-ai-pypi-supply-chain-attack-mistralai-246-what-python-ai-developers-must-do-right-now-c8i
- guardrails-ai PyPI: https://thehackernews.com/2026/05/mini-shai-hulud-worm-compromises.html

## 2026.05.12.1

Pattern update 2026-05-12. One new rule: MAL-081 (Mini Shai-Hulud wave-3 TeamPCP supply-chain attack on @mistralai/@tanstack npm packages).

- **MAL-081** (critical, new): **Mini Shai-Hulud wave-3 (TeamPCP) supply-chain attack on @mistralai/@tanstack npm packages via CI/CD cache poisoning (CVE-2026-45321, GHSA-g7cv-rxg3-hmpx)** — On 2026-05-11, TeamPCP published 84 malicious npm artifacts across 42 @tanstack packages and compromised @mistralai/mistralai 2.2.2-2.2.4, @mistralai/mistralai-azure 1.7.1-1.7.3, @mistralai/mistralai-gcp 1.7.1-1.7.3 (GHSA-3q49-cfcf-g5fm / MAL-2026-3432, CVSS 9.6). Attack chained a pull_request_target Pwn Request misconfiguration with GitHub Actions cache poisoning (1.1 GB entry) and /proc/<pid>/mem OIDC token extraction to publish malicious versions via the project's own trusted release workflow — the first documented supply-chain attack producing artifacts with valid SLSA provenance. Payload: preinstall hook downloads Bun, executes ~2.3 MB obfuscated binary, installs gh-token-monitor daemon (macOS LaunchAgent / Linux systemd) polling GitHub every 60 s. Exfil via typosquat C2 `git-tanstack.com`, Session messenger (`filev2.getsession.org`, `seed1-3.getsession.org`), and GitHub API dead drops. Secondary payload: `/tmp/transformers.pyz`. Mutex: `/tmp/tmp.ts018051808.lock`. The existing TeamPCP rules (MAL-039, MAL-041, MAL-049) cover prior waves (Trivy, Checkmarx, SAP CAP) with their respective IOCs; this rule anchors on the new-wave-specific C2 and persistence indicators.

Vuln DB additions: `npm/@mistralai/mistralai` 2.2.2-2.2.4 (GHSA-3q49-cfcf-g5fm, critical, REMOVED), `npm/@mistralai/mistralai-azure` 1.7.1-1.7.3 (same), `npm/@mistralai/mistralai-gcp` 1.7.1-1.7.3 (same). 9 package-version entries total.
IOC additions: 1 domain (`git-tanstack.com`).

Total: 287 static rules + 14 chain rules = 301.

Sources:
- StepSecurity analysis: https://www.stepsecurity.io/blog/mini-shai-hulud-is-back-a-self-spreading-supply-chain-attack-hits-the-npm-ecosystem
- TanStack GitHub issue: https://github.com/TanStack/router/issues/7383
- Mistral AI GitHub issue: https://github.com/mistralai/client-ts/issues/217
- The Hacker News: https://thehackernews.com/2026/05/mini-shai-hulud-worm-compromises.html
- Aikido Security: https://www.aikido.dev/blog/mini-shai-hulud-is-back-tanstack-compromised

Candidates researched and already covered or excluded: CVE-2026-20205 Splunk MCP (already covered, 2026.04.XX), GHSA-MJ59-H3Q9-GHFH OpenClaw MCP stdio env var injection (already covered as PSV rule), CVE-2026-43901 Wireshark MCP (medium severity, no AI-toolchain risk angle), CVE-2026-30635 (inaccessible source, insufficient anchors), prior TeamPCP waves (MAL-039/MAL-041/MAL-049 cover Trivy/Checkmarx/SAP waves).

## 2026.05.11.1

Pattern update 2026-05-11. Two new rules: PSV-074 (Cline kanban WebSocket hijacking, CVE-2026-44211), PSV-075 (n8n-MCP log disclosure, CVE-2026-41495).

- **PSV-074** (critical, new): **Cline kanban server cross-origin WebSocket hijacking, no Origin validation (CVE-2026-44211, GHSA-5c57-rqjx-35g2, cline <= 2.13.0)** — Disclosed by Oasis Security on May 8, 2026 (CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:H Critical, CWE-1385/CWE-306). The Cline kanban agent-orchestration board (npm package `cline`, all versions up to 2.13.0) starts a WebSocket server on 127.0.0.1:3484 with no Origin header validation. Because browsers do not enforce CORS on WebSocket handshakes, any web page a developer visits while kanban is running can silently connect to port 3484 and (1) exfiltrate real-time workspace data (filesystem paths, task descriptions, git branch names, AI agent chat messages) or (2) inject arbitrary prompts into running Cline/Claude Code/Codex agent sessions, enabling RCE. No patch available as of disclosure. Rule fires on CVE ID, GHSA ID, cline + kanban + websocket/origin/port-3484 context, and port :3484 + origin-validation keywords.

- **PSV-075** (low, new): **n8n-MCP HTTP transport logs bearer tokens from unauthorized /mcp requests (CVE-2026-41495, n8n-mcp < 2.47.11)** — CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N Low, CWE-532. When czlonkowski/n8n-mcp runs in HTTP transport mode, POST /mcp request metadata — bearer tokens, per-tenant API keys, and JSON-RPC payloads — is written to server logs regardless of authentication outcome. Unauthenticated requests are correctly rejected with 401, but the rejected request headers (including credentials) persist in logs. In deployments with centralized log aggregation, SIEM pipelines, or shared log storage, this exposes authentication tokens to parties outside the request trust boundary. Fixed in n8n-mcp 2.47.11. Rule fires on CVE ID, n8n-mcp + bearer/API key + log persistence keywords, and czlonkowski/n8n-mcp + log/sensitive/bearer context.

Vuln DB additions: `n8n-mcp` (CVE-2026-41495, low, < 2.47.11).
IOC additions: none.

Total: 286 static rules + 14 chain rules = 300.

Sources:
- CVE-2026-44211 (GitLab Advisory): https://advisories.gitlab.com/npm/cline/CVE-2026-44211/
- CVE-2026-44211 (Oasis Security): https://www.oasis.security/blog/cline-kanban-websocket-hijack
- CVE-2026-44211 (GBHackers): https://gbhackers.com/cline-kanban-websocket-vulnerability/
- CVE-2026-41495 (GitLab Advisory): https://advisories.gitlab.com/npm/n8n-mcp/CVE-2026-41495/
- CVE-2026-41495 (OffSeq Radar): https://radar.offseq.com/threat/cve-2026-41495-cwe-532-insertion-of-sensitive-info-dc23ff94

Candidates researched and already covered or excluded: CVE-2026-44895 @yoda.digital/gitlab-mcp-server (PSV-072, covered 2026.05.10.1), GHSA-v6wj-c83f-v46x @profullstack/mcp-server (PSV-073, covered 2026.05.10.1), CVE-2026-44211 Cline kanban WebSocket (new PSV-074), CVE-2026-41495 n8n-MCP log disclosure (new PSV-075), CVE-2026-33032 nginx-ui (covered), CVE-2026-20205 Splunk MCP (covered), CVE-2026-33980 Azure Data Explorer MCP (covered), CVE-2026-26118 Azure MCP SSRF (covered), PyTorch Lightning Mini Shai-Hulud (covered), QLNX Quasar Linux (covered MAL-080), intercom-client 7.0.4/7.0.5 (covered), @bitwarden/cli Shai-Hulud (covered), axios 1.14.1/0.30.4 (covered), CVE-2026-39974 n8n-MCP SSRF (covered PSV-018), CVE-2026-5058 aws-mcp-server (covered), CVE-2026-32211 Azure DevOps MCP (covered), CVE-2026-30615 Windsurf zero-click (covered), TrustFall enableAllProjectMcpServers (covered), MCPJam Inspector CVE-2026-23744 (covered), CVE-2025-65717 Live Server (covered), LiteLLM TeamPCP (covered), CVE-2026-44284 FastGPT SSRF (PSV-071, covered).

## 2026.05.10.1

Pattern update 2026-05-10. Three new rules: PINJ-028 (Spring AI memory poisoning), PSV-072 (GitLab MCP SSE auth bypass), PSV-073 (profullstack MCP command injection).

- **PINJ-028** (high, new): **Spring AI PromptChatMemoryAdvisor prompt injection via memory poisoning (CVE-2026-41713, spring-ai < 1.0.7 / < 1.1.6)** — Published May 8, 2026 (CVSS 7.2 HIGH, CWE-99). The PromptChatMemoryAdvisor component in Spring AI 1.0.0–1.0.6 and 1.1.0–1.1.5 stores user input in conversation memory without sanitization; that stored content is later reinterpreted by the AI model, enabling cross-turn prompt injection. PromptChatMemoryAdvisor has been deprecated; the fix requires callers to supply an explicit conversation ID. Fixed in spring-ai 1.0.7 and 1.1.6. Rule fires on CVE ID, `PromptChatMemoryAdvisor` class references, and Maven artifact `org.springframework.ai:*` at vulnerable version strings. Reporter: Ahmed Sekka.

- **PSV-072** (high, new): **@yoda.digital/gitlab-mcp-server SSE transport auth bypass and wildcard CORS (CVE-2026-44895, < 0.6.0)** — All versions of @yoda.digital/gitlab-mcp-server before 0.6.0 run their SSE HTTP transport with no authentication and wildcard CORS on all interfaces (0.0.0.0). `GET /sse` and `POST /messages?sessionId=<id>` require no credentials; any network-adjacent attacker can open an SSE session and issue arbitrary MCP tool calls using the server's loaded `GITLAB_PERSONAL_ACCESS_TOKEN`, gaining full access to all 86 GitLab tools. CVSS HIGH (C:H/I:N/A:H), CWE-306 + CWE-942. Fixed in 0.6.0. Rule fires on CVE ID, npm package name `@yoda.digital/gitlab-mcp-server`, and GitHub repo `yoda-digital/mcp-gitlab-server`.

- **PSV-073** (critical, new): **@profullstack/mcp-server unauthenticated OS command injection in domain_lookup (GHSA-v6wj-c83f-v46x, <= 1.4.12, no fix)** — CVSS 9.8 CRITICAL (CWE-78). The domain_lookup module concatenates user-supplied `domains`/`keywords` arrays directly into shell commands via `execAsync()` (pattern: `tldx ${keywords.join(' ')}`). Shell metacharacters in input execute arbitrary commands with server process privileges. The server binds to 0.0.0.0 with no authentication; exploitation is remote and unauthenticated. No fix available as of 2026-05-10. Rule fires on GHSA ID and npm package name `@profullstack/mcp-server`.

Vuln DB additions: `spring-ai` (CVE-2026-41713, high, 1.0.0–1.0.6 / 1.1.0–1.1.5), `@yoda.digital/gitlab-mcp-server` (CVE-2026-44895, high, < 0.6.0), `@profullstack/mcp-server` (GHSA-v6wj-c83f-v46x, critical, <= 1.4.12).
IOC additions: none.

Total: 284 static rules + 14 chain rules = 298.

Sources:
- CVE-2026-41713 (Spring): https://spring.io/security/cve-2026-41713/
- Spring AI 1.0.7/1.1.6 release: https://spring.io/blog/2026/05/08/spring-ai-1-0-7-1-1-6-2-0-0-M6-available-now/
- CVE-2026-44895 (GitLab advisory): https://advisories.gitlab.com/npm/@yoda.digital/gitlab-mcp-server/CVE-2026-44895/
- GHSA-v6wj-c83f-v46x (GitHub): https://github.com/advisories/GHSA-v6wj-c83f-v46x

Candidates researched and already covered or excluded: Splunk MCP CVE-2026-20205 (covered), Azure Data Explorer MCP CVE-2026-33980 (covered), PyTorch Lightning Mini Shai-Hulud (covered), QLNX Quasar Linux RAT (covered MAL-080), GlassWorm (covered MAL-066 + multiple waves).

## 2026.05.09.1

Pattern update 2026-05-09. One new rule: PSV-071 (FastGPT MCP tool endpoint SSRF, CVE-2026-44284).

- **PSV-071** (medium, new): **FastGPT MCP tool endpoint SSRF via stored internal URL (CVE-2026-44284, FastGPT < 4.14.17)** — Disclosed May 8, 2026 (CVSS 6.3, CWE-918). Prior to version 4.14.17, FastGPT had an inconsistent SSRF protection gap in MCP tool URL handling. An authenticated user with permission to create or manage MCP toolsets could store an internal endpoint (e.g. `http://localhost:3000/mcp`) as an MCP tool URL; the FastGPT backend workflow runner later fetches that stored URL without revalidating the destination, bypassing SSRF protection. This allows an authenticated attacker to pivot to internal services reachable by the FastGPT backend. Fixed in FastGPT 4.14.17. Rule fires on CVE ID, FastGPT + internal URL near MCP tool/endpoint keywords, and FastGPT + version strings in the vulnerable range (< 4.14.17).

Vuln DB additions: `fastgpt` (CVE-2026-44284, medium, < 4.14.17).
IOC additions: none.

Total: 281 static rules + 14 chain rules = 295.

Sources:
- CVE-2026-44284 (TheHackerWire): https://www.thehackerwire.com/vulnerability/CVE-2026-44284/
- CVE-2026-44284 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-44284

Candidates researched and already covered or excluded: QLNX Quasar Linux RAT (MAL-080, 2026.05.07.1), OpenClaw log poisoning GHSA-g27f-9qjv-22pm (PINJ-027, 2026.05.08.1), Splunk MCP CVE-2026-20205 (covered), Azure Data Explorer MCP CVE-2026-33980 (covered), n8n-MCP SSRF CVE-2026-39974 (covered), OpenClaw auth bypass CVE-2026-22172 (covered), PyTorch Lightning Mini Shai-Hulud (covered).

## 2026.05.08.1

Pattern update 2026-05-08. One new rule: PINJ-027 (OpenClaw log poisoning — indirect prompt injection via WebSocket headers, GHSA-g27f-9qjv-22pm).

- **PINJ-027** (medium, new): **OpenClaw log poisoning — indirect prompt injection via WebSocket headers (GHSA-g27f-9qjv-22pm, openclaw < 2026.2.13)** — Eye Security disclosed (GHSA-g27f-9qjv-22pm) that in OpenClaw prior to 2026.2.13, the gateway logs raw Origin and User-Agent WebSocket header values on the "closed before connect" path without sanitization or length limits. An unauthenticated attacker can send crafted connection attempts with up to ~15 KB of arbitrary header content, which is written verbatim into core gateway logs. Under AI-assisted debugging workflows where those logs are fed to an LLM, the injected content acts as an indirect prompt injection payload — potentially hijacking the LLM's actions. Fixed in 2026.2.13 (header values now sanitized and truncated before logging). Rule fires on GHSA ID, openclaw + log-poison context, and openclaw version strings in the vulnerable range (< 2026.2.13).

Vuln DB additions: `openclaw` (GHSA-g27f-9qjv-22pm, medium, < 2026.2.13).
IOC additions: none.

Total: 280 static rules + 14 chain rules = 294.

Sources:
- GHSA-g27f-9qjv-22pm (GitHub Advisory): https://github.com/openclaw/openclaw/security/advisories/GHSA-g27f-9qjv-22pm
- Eye Security research: https://research.eye.security/log-poisoning-in-openclaw/
- Eye Security blog: https://www.eye.security/blog/log-poisoning-openclaw-ai-agent-injection-risk

Candidates researched and already covered or excluded: QLNX Quasar Linux RAT (MAL-080, 2026.05.07.1), Oracle MCP Server Helper CVE-2026-35228 (PSV-070, 2026.05.07.1), n8n-MCP SSRF CVE-2026-39974 (covered), Azure Data Explorer MCP KQL injection CVE-2026-33980 (covered), Splunk MCP CVE-2026-20205 (covered), PyTorch Lightning Mini Shai-Hulud (covered), SAP CAP npm packages (SUP-049, covered), Axios npm compromise (covered), LiteLLM TeamPCP (covered).

## 2026.05.07.1

Pattern update 2026-05-07. Two new rules: MAL-080 (QLNX Quasar Linux RAT) and PSV-070 (Oracle MCP Server Helper Tool SQL injection, CVE-2026-35228).

- **MAL-080** (critical, new): **QLNX (Quasar Linux) PAM backdoor and developer credential harvester targeting supply chain** — Trend Micro disclosed QLNX (May 5, 2026), a previously undocumented Linux RAT targeting developer and DevOps workstations to enable downstream supply chain attacks. QLNX deploys a malicious PAM module via LD_PRELOAD compiled on the target host from embedded C source, intercepting plaintext passwords at authentication time and XOR-encrypting them to `/var/log/.ICE-unix`. A dual-layer rootkit (userland LD_PRELOAD + kernel-level eBPF) hides files, PIDs, and network ports. The credential harvester extracts secrets from `.npmrc`, `.pypirc`, `.git-credentials`, `.aws/credentials`, `.kube/config`, `.docker/config.json`, `.vault-token`, and `.env`, then exfiltrates via a 58-command TLS C2 framework. Stolen registry tokens are used for downstream package poisoning. Distinctive hard IOCs: hardcoded PAM master password `O$$f$QtYJK`; credential cache at `/var/log/.ICE-unix`; lock file `/tmp/.X752e2ca1-lock` (DJB2("quasar_linux") = 0x752e2ca1). The LiteLLM supply chain compromise (March 2026) followed this exact credential-theft-to-trojanization pattern.
- **PSV-070** (high, new): **Oracle MCP Server Helper Tool unauthenticated SQL injection via HTTP (CVE-2026-35228, versions 1.0.1 – 1.0.156)** — Oracle Critical Patch Update (April 2026) discloses CVE-2026-35228, an easily exploitable SQL injection vulnerability in the Oracle MCP Server Helper Tool (Oracle Open Source Projects). An unauthenticated attacker with HTTP network access can submit crafted requests that cause the tool to execute malicious SQL, enabling unauthorized data access or modification. Affects versions 1.0.1 through 1.0.156; update above 1.0.156.

Vuln DB additions: `oracle-mcp-server-helper` (CVE-2026-35228, high, versions 1.0.1–1.0.156).
IOC additions: none.

Total: 279 static rules + 14 chain rules = 293.

Sources:
- QLNX (Trend Micro): https://www.trendmicro.com/en_us/research/26/e/quasar-linux-qlnx-a-silent-foothold-in-the-software-supply-chain.html
- QLNX (BleepingComputer): https://www.bleepingcomputer.com/news/security/new-stealthy-quasar-linux-malware-targets-software-developers/
- QLNX (SOC Prime): https://socprime.com/active-threats/qlnx-linux-rat-uses-rootkit-and-pam-backdoor/
- CVE-2026-35228 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-35228
- CVE-2026-35228 (Oracle CPUApr2026): https://www.oracle.com/security-alerts/cpuapr2026.html

Candidates researched and already covered or excluded: PyTorch Lightning Mini Shai-Hulud (existing MAL+SUP rules + vuln_db), CanisterSprawl (existing MAL rule + vuln_db), n8n-MCP SSRF CVE-2026-39974 (PSV rule + vuln_db), Azure Data Explorer MCP KQL injection CVE-2026-33980 (PSV rule + vuln_db), Splunk MCP CVE-2026-20205 (PSV rule + vuln_db), GlassWorm (multiple existing MAL rules), Live Server CVE-2025-65717 (existing PSV rule + vuln_db), MCP STDIO systemic RCE variants (existing PSV rules for individual CVEs).

## 2026.05.06.1

Pattern update 2026-05-06. Three new PSV rules:

- **PSV-067** — `mcp-atlassian` MCPwnfluence: unauthenticated SSRF + arbitrary file write RCE chain (CVE-2026-27825 CVSS 9.1 + CVE-2026-27826 CVSS 8.2, GHSA-xjgw-4wvw-rgm4, < 0.17.0, 4M+ downloads). `confluence_download_attachment` tool accepts unconstrained `download_path` (path traversal to authorized_keys/cron.d); `X-Atlassian-Jira-Url` / `X-Atlassian-Confluence-Url` headers honored without validation (SSRF). Pluto Security "MCPwnfluence" disclosure 2026-02-19; fixed 2026-02-24.
- **PSV-068** — MCP Java SDK (`io.modelcontextprotocol.sdk:mcp-core`) DNS rebinding via missing Origin validation (CVE-2026-35568, GHSA-8jxr-pr72-r468, CVSS 7.6, CWE-346, < 1.0.0). No Origin header validation before 1.0.0 allows browser-based DNS rebinding to invoke any MCP tool call on a local server. Fixed in 1.0.0 (2026-04-07).
- **PSV-069** — OpenClaw MCP stdio config env var injection via unsafe cast in `resolveStdioMcpServerLaunchConfig` (GHSA-MJ59-H3Q9-GHFH, < 2026.4.20). Opening a crafted workspace config triggers arbitrary code execution via malicious env entries (NODE_OPTIONS, LD_PRELOAD). Distinct from CVE-2026-22177 (PSV-050, < 2026.2.21) which targeted general startup config. Fixed 2026-04-20.

Vuln DB additions: `mcp-atlassian` (CVE-2026-27825, CVE-2026-27826), `mcp-java-sdk` (CVE-2026-35568), `openclaw` (GHSA-MJ59-H3Q9-GHFH).

## 2026.05.05.1

Pattern update 2026-05-05. Three new rules: SUP-051 (dYdX supply chain compromise — wallet stealer + RAT), PSV-065 (xcode-mcp-server OS command injection via CVE-2026-7416), PSV-066 (hwpx-mcp path traversal via CVE-2026-7599). Vuln DB additions for @dydxprotocol/v4-client-js (4 npm versions), dydx-v4-client (PyPI), xcode-mcp-server, and hwpx-mcp. IOC addition: dydx.priceoracle.site.

- **SUP-051** (high, new): **dYdX @dydxprotocol/v4-client-js supply chain compromise — wallet stealer + RAT (C2: dydx.priceoracle.site, January 2026)** — Four npm versions (3.4.1, 1.22.1, 1.15.2, 1.0.31) and PyPI package dydx-v4-client 1.1.5post1 were published via stolen dYdX developer credentials and identified by Socket on January 27, 2026. Malicious code injected into registry.ts harvests cryptocurrency wallet seed phrases and device fingerprints, exfiltrating them to the attacker-controlled C2 dydx.priceoracle.site/v4/price. The PyPI variant additionally drops a Remote Access Trojan with C2 retrieval via dydx.priceoracle.site/py. Remove affected packages, upgrade to clean versions, and rotate all wallet seed phrases and developer credentials.
- **PSV-065** (high, new): **xcode-mcp-server OS command injection via build_project/run_tests (CVE-2026-7416, PolarVista, 1.0.0, CVSS 7.3 HIGH)** — PolarVista/xcode-mcp-server version 1.0.0 passes the user-supplied Request argument to OS command execution without sanitization in the build_project and run_tests MCP tool handlers (src/index.ts). Exploitable remotely without authentication (AV:N/AC:L/PR:N/UI:N). No patch was available at time of disclosure (May 1, 2026). Avoid deploying this MCP server on internet-accessible hosts.
- **PSV-066** (medium, new): **hwpx-mcp path traversal via output_path in document export tools (CVE-2026-7599, Dayoooun, 0.2.0, CVSS 6.3 MEDIUM, CWE-22)** — Dayoooun/hwpx-mcp version 0.2.0, an MCP server for Korean HWP/HWPX document formats, accepts a user-supplied output_path parameter in save_document, export_to_text, and export_to_html without sanitization, enabling arbitrary file write via path traversal sequences. Disclosed May 1, 2026; no patch available.
- **Vuln DB additions:** npm/@dydxprotocol/v4-client-js 3.4.1/1.22.1/1.15.2/1.0.31 → NPM-BACKDOOR-2026-0127 (critical). python/dydx-v4-client 1.1.5post1 → PYPI-BACKDOOR-2026-0127 (critical). Product entries: xcode-mcp-server 1.0.0 → CVE-2026-7416 (high, no fix). hwpx-mcp 0.2.0 → CVE-2026-7599 (medium, no fix).
- **IOC additions:** dydx.priceoracle.site (dYdX supply chain C2, January 2026).

Candidates researched and already covered: PyTorch Lightning Mini Shai-Hulud (MAL + SUP-050), CanisterSprawl/pgserve (CanisterWorm rules + Namastex rule), n8n-MCP SSRF CVE-2026-39974 (PSV), Azure Data Explorer MCP KQL injection CVE-2026-33980 (PSV), Splunk MCP CVE-2026-20205 (PSV), magento2-dev-mcp CVE-2026-5603 (PSV), command-executor-mcp-server CVE-2026-7593 (PSV-062), XInference PyPI (existing rule), axios npm supply chain (existing rule), intercom-client supply chain (existing rule), SAP CAP npm Mini Shai-Hulud (existing rule), LiteLLM compromise (existing rule), GlassWorm/OpenVSX (existing rules).

Total: 274 static rules + 14 chain rules = 288.

Sources:
- dYdX supply chain (Socket): https://socket.dev/blog/malicious-dydx-packages-published-to-npm-and-pypi
- dYdX supply chain (THN): https://thehackernews.com/2026/02/compromised-dydx-npm-and-pypi-packages.html
- CVE-2026-7416 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-7416
- CVE-2026-7416 (ThreatInt): https://cve.threatint.eu/CVE/CVE-2026-7416
- CVE-2026-7599 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2026-7599
- CVE-2026-7599 (ThreatInt): https://cve.threatint.eu/CVE/CVE-2026-7599

## 2026.05.04.1

Pattern update 2026-05-04. One new PSV rule (CVE-2025-54994 @akoskm/create-mcp-server-stdio command injection). Enrichment for SUP-039 (intercom-client 7.0.5). Vuln DB addition for intercom-client 7.0.4/7.0.5. YAML structural fix: MAL-078, MAL-079, and SUP-050 were committed in rulepack 2026.05.03.2 with incorrect placement in chain_rules (wrong indentation, missing metadata); moved to static_rules with full metadata blocks and corrected test inputs.

- **PSV-064** (critical, new): **@akoskm/create-mcp-server-stdio MCP tool command injection via exec() (CVE-2025-54994, GHSA-3ch2-jxxc-v4xf, CVSS 9.3, CWE-77/CWE-78, < 0.0.13)** — The MCP server's built-in `which-app-on-port` tool concatenates user-supplied port numbers directly into `exec('lsof -i :<port>')` without sanitization. An attacker who can influence the port argument — via prompt injection targeting an AI agent using this MCP server — can inject shell metacharacters to execute arbitrary OS commands on the host. Disclosed by Liran Tal (September 2025); highlighted in the OX Security MCP supply chain advisory (April 2026). Upgrade to @akoskm/create-mcp-server-stdio 0.0.13 or later, which replaces exec() with execFile() and argv arrays.
- **SUP-039 enrichment**: title and pattern updated to cover intercom-client 7.0.5 (additionally confirmed compromised by Wiz alongside 7.0.4 in the Mini Shai-Hulud/TeamPCP campaign). Technique name corrected from placeholder to "Supply Chain Compromise via Package Manager". References added.
- **MAL-078 fix** (Mini Shai-Hulud Bun Runtime Loader): moved from chain_rules to static_rules, added full metadata block, expanded mitigation.
- **MAL-079 fix** (Mini Shai-Hulud IDE persistence hook): moved from chain_rules to static_rules, added full metadata block, fixed multi-line test_input to single-line (regex `.` does not cross newlines without multiline flag), expanded title and mitigation.
- **SUP-050 fix** (Compromised PyTorch Lightning version reference): moved from chain_rules to static_rules, added full metadata block, fixed test_input from YAML `:` notation to pip `==` notation so the pattern actually fires.
- **Vuln DB additions:** `npm/intercom-client` 7.0.4 and 7.0.5 → PYPI-BACKDOOR-2026-0430 (critical, fixed: 7.0.6).

Note: CHANGELOG entry for rulepack version 2026.05.03.2 is missing (rules were merged without a changelog entry). Separate cleanup PR needed per SKILL.md convention.

Total: 270 static rules + 14 chain rules = 284.

Sources:
- CVE-2025-54994 (GHSA): https://github.com/advisories/GHSA-3ch2-jxxc-v4xf
- CVE-2025-54994 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2025-54994
- Node.js security advisory: https://www.nodejs-security.com/blog/create-mcp-server-stdio-command-injection-vulnerability
- OX Security MCP advisory: https://www.ox.security/blog/mcp-supply-chain-advisory-rce-vulnerabilities-across-the-ai-ecosystem/
- intercom-client 7.0.5 (Wiz): https://www.wiz.io/blog/mini-shai-hulud-supply-chain-sap-npm
- intercom-client 7.0.4 (Socket): https://socket.dev/blog/intercom-s-npm-package-compromised-in-supply-chain-attack

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
