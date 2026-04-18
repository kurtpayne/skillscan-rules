# SkillScan Rules

Detection rules, IOC database, and vulnerability database for [SkillScan](https://skillscan.sh).

This repo is the source of truth for SkillScan's detection content. The scanner fetches updates from here via `skillscan update`.

## Contents

- `rules/default.yaml` — Static detection rules (220+ rules across 12 categories)
- `rules/ast_flows.yaml` — Python AST data-flow taint analysis rules
- `rules/multilang.yaml` — Language-specific rules (.js, .ts, .rb, .go, .rs)
- `intel/ioc_db.json` — Indicators of compromise (domains, IPs, CIDRs)
- `intel/vuln_db.json` — Vulnerable package versions (Python + npm)
- `intel/managed_sources.json` — External intel feed configuration
- `manifest.json` — Version and SHA-256 hashes for all files

## Updating

SkillScan users get the latest rules automatically:

```bash
skillscan update           # Update rules + intel + model
skillscan update --rules-only  # Just rules and intel (fast)
```

## Contributing

To propose a new detection rule:

1. Fork this repo
2. Add your rule to `rules/default.yaml`
3. Update `manifest.json` (bump version, recalculate hashes)
4. Open a PR

See [SkillScan's custom rules format](https://github.com/kurtpayne/skillscan-security/blob/main/docs/custom-rules-format.md) for the rule schema.

## License

Apache-2.0
