# Changelog

All notable changes to `first-look` are documented in this file.

## [1.0.0] - 2026-11-22

Initial public release of the `first-look` Agent Skill.

### Added
- **Two-mode architecture** (`SKILL.md`): read-only **Light mode** (default pre-release delivery & exposure audit) and sandboxed **Deep mode** (explicit cold-start run verification).
- **Compact 4-block Light report**: scope-locked 1-sentence verdict, 5-layer delivery boundary table, risk-ranked external exposure table with explicit direct-fact vs `[inference]` reader takeaways, and public-claims guardrails.
- **Portable read-only inspection playbook** (`references/checklist.md`): hardened Git commands (`GIT_OPTIONAL_LOCKS=0`, disabled `fsmonitor`/hooks/prompts), source-level redaction before stdout/tool logs, full reachable-history checks (author + committer + deleted files), non-Git/archive/wheel/OCI/LFS/submodule/multi-license coverage, and Deep-mode container/VM isolation rules.
- **Calibrated examples & anti-patterns** (`references/examples.md`): Light + Deep output samples and guardrails against prose-style guessing, old-context negation, static-vs-executed conflation, and raw secret echoing.
- **Bilingual triggers & CN/EN label map** for seamless use in both English and Chinese workflows.
