# Changelog

All notable changes to `first-look` are documented in this file.

## [1.1.0] - 2026-11-28

### Added
- **Dual-perspective audit routing (`SKILL.md`, `references/checklist.md` §4A/§4B, `references/examples.md`)**: separates **Research / Model lens** (weights, datasets, runtime layers, academic vs. commercial licenses, hard stops) from **Public Tool / Skill UX lens** (10-second first-visit clarity, zero-config onboarding, generalizability without host lock-in, doc-vs-code parity). If the caller does not specify a lens, the skill asks 1 quick routing question first.

### Changed
- **Concise `SKILL.md` frontmatter**: tightened `description` with bilingual triggers and explicit non-goals for clean skill discovery.

### Fixed
- **Portable shell snippets in `references/checklist.md`**: replaced the `$GIT_SAFE` string prefix with a `git_safe()` shell function so commands behave identically under `sh`, `bash`, and interactive `zsh` (zsh does not word-split unquoted variables).
- **macOS-compatible remote probe**: stock macOS lacks GNU `timeout`; the anonymous `ls-remote` probe now prefers `timeout` when present and falls back to `perl -e 'alarm shift; exec @ARGV'`.
- **Safe manifest temp file**: archive listing scan now uses `mktemp "${TMPDIR:-/tmp}/fl_manifest.XXXXXX"` inside a subshell with an `EXIT` trap cleanup instead of a hardcoded `/tmp/fl_manifest.txt`, avoiding concurrent collisions; the subshell sets `setopt null_glob` under `zsh` so unmatched `dist/*.tar.gz` patterns skip instead of aborting the loop.
- **Honest `grep` exit codes**: the annotated-tag scan and manifest pattern scan no longer use `|| true`; `grep` exit code `1` (no match) is treated as clean while `>=2` emits `SCAN_ERROR`, consistent with the stated error-handling rule.

## [1.0.0] - 2026-11-22

Initial public release of the `first-look` Agent Skill.

### Added
- **Two-mode architecture** (`SKILL.md`): read-only **Light mode** (default pre-release delivery & exposure audit) and sandboxed **Deep mode** (explicit cold-start run verification).
- **Compact 4-block Light report**: scope-locked 1-sentence verdict, 5-layer delivery boundary table, risk-ranked external exposure table with explicit direct-fact vs `[inference]` reader takeaways, and public-claims guardrails.
- **Portable read-only inspection playbook** (`references/checklist.md`): hardened Git commands (`GIT_OPTIONAL_LOCKS=0`, disabled `fsmonitor`/hooks/prompts), source-level redaction before stdout/tool logs, full reachable-history checks (author + committer + deleted files), non-Git/archive/wheel/OCI/LFS/submodule/multi-license coverage, and Deep-mode container/VM isolation rules.
- **Calibrated examples & anti-patterns** (`references/examples.md`): Light + Deep output samples and guardrails against prose-style guessing, old-context negation, static-vs-executed conflation, and raw secret echoing.
- **Bilingual triggers & CN/EN label map** for seamless use in both English and Chinese workflows.
