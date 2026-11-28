<p align="center">
  <img src="assets/hero.png" alt="first-look — Pre-release Repo Audit for AI Agents" width="100%" />
</p>

<h1 align="center">first-look</h1>

<p align="center">
  <strong>A pre-release repository audit &amp; cold-start check skill for AI coding agents.</strong><br />
  See your repository through a first-time visitor's eyes: what it appears to be, what it actually ships vs. misses, and what unintended traces are already exposed.
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-0EA5E9.svg?style=flat-square" alt="MIT License" /></a>
  <a href="https://agentskills.io/"><img src="https://img.shields.io/badge/Agent_Skills-compatible-10B981.svg?style=flat-square" alt="Agent Skills Compatible" /></a>
  <a href="CHANGELOG.md"><img src="https://img.shields.io/badge/version-1.1.0-38BDF8.svg?style=flat-square" alt="Version 1.1.0" /></a>
</p>

<p align="center">
  <strong>English</strong> | <a href="README.zh-CN.md">简体中文</a>
</p>

When you know a repository inside out, it is surprisingly easy to ship hidden assumptions ("one command runs everything" when a model weight or separate runtime environment is missing) or unintended traces (personal commit emails, internal project codenames, draft roadmaps, or leftover agent files).

**`first-look`** gives any Agent Skills–compatible coding agent a disciplined, read-only **external-visitor lens** before you publish a repository or whenever you inspect an unfamiliar one.

---

## Two Lenses × Two Depths

If you do not specify which type of repository is being audited, the agent asks one quick routing question first:

| Lens | Focus | Output focus |
| --- | --- | --- |
| **Research / Model** | Models, paper implementations, scientific libraries | 5-layer delivery table per stage (Code, Runtime, Weights/Data, Licenses/Hardware, Network/Credentials) and hard reproduction blockers. |
| **Public Tool / Skill UX** | CLIs, libraries, Agent Skills, user-facing apps | 3-block first-visit walkthrough (1-sentence verdict, 4-step real onboarding flow with exact friction points, actionable pre-release gaps & privacy summary without noisy filler). |

| Depth | Trigger | What it does |
| --- | --- | --- |
| **Light (default)** | Any normal first-look / pre-release check | **Read-only.** Never installs packages or downloads large files. Outputs 4 compact blocks: **1-sentence verdict**, **delivery / UX table**, **risk-ranked external exposure table**, and **public claims guardrail**. |
| **Deep (explicit)** | Only when you explicitly ask to install & run | Runs the actual release artifact inside a **disposable sandbox** (empty `HOME`, stripped credentials, verified archive paths) and binds every `Executed` verdict to the exact channel and digest. Refuses to execute on the host if isolation is unavailable. |

### What Light Mode Audits
- **Delivery boundaries or Tool onboarding UX**: structured per-stage check depending on the selected lens ([`references/checklist.md`](references/checklist.md) §4A / §4B).
- **External exposure (3 visibility tiers)**: Separates `Local only`, `Planned public` (tracked tree, full reachable Git history, archives/packages), and `Already public` (remote/CI/releases/forks).
  - Source-level redaction: scanners never dump raw secrets, personal emails, or local usernames into tool logs.
  - Evidence-only AI/agent detection: reports concrete files/lines neutrally (never guesses authorship from prose style, and never flags intentionally published `AGENTS.md`/`CLAUDE.md` as defects by default).
  - Multi-license & third-party redistribution checks: separates **downloadable**, **usable**, and **redistributable**.

---

## Installation

Follows the standard [Agent Skills](https://agentskills.io/) layout (`SKILL.md` + `references/`). Compatible with Claude Code, Codex, Cursor, OpenCode, Oh My Pi, and other Agent Skills hosts.

### Option 1 — Skills CLI
```sh
npx skills add SeanYuanWSY/first-look
```

### Option 2 — Git clone (global or project-local)
```sh
# Universal shared skills directory (Codex, Oh My Pi, etc.)
git clone https://github.com/SeanYuanWSY/first-look.git ~/.agents/skills/first-look

# Or Claude Code personal skills directory
git clone https://github.com/SeanYuanWSY/first-look.git ~/.claude/skills/first-look
```

### Uninstall
```sh
rm -rf ~/.agents/skills/first-look   # or ~/.claude/skills/first-look
```

> **Platform note**: Shell snippets in [`references/checklist.md`](references/checklist.md) target POSIX shells (macOS / Linux); Windows users can use the noted PowerShell equivalents or tool-neutral steps.

---

## Usage Examples

- *"Run a `first-look` check on this research repo before publishing — focus on weights, dataset requirements, and reproduction blockers."*
- *"Use `first-look` from a public-tool UX lens to check whether a first-time visitor understands our CLI/Skill in 10 seconds and exposes any private traces."*
- *"Run `first-look` in **deep mode** on `dist/mypkg-1.0.0.tar.gz` in a clean container."*

The agent replies in the language of your prompt using consistent English or Chinese status labels.

---

## Scope & Non-Goals

- **Read-only by default**: inspects and recommends; never edits files, rewrites Git history, or publishes on its own.
- **Not a replacement** for unit tests, code review, penetration testing, or legal compliance sign-off.

---

## Repository Layout

- [`SKILL.md`](SKILL.md) — Core skill specification (modes, observer isolation, 4-block schema, deep-mode safety contract, CN/EN label map)
- [`references/checklist.md`](references/checklist.md) — Safe read-only commands, source-level redaction checks, non-Git/archive/OCI/LFS/multi-license rules, and sandbox isolation rules
- [`references/examples.md`](references/examples.md) — Calibrated Light & Deep output examples plus common anti-patterns
- [`assets/`](assets/) — Repository hero banner ([`hero.png`](assets/hero.png), 1280×640, GitHub Social Preview ready) and square icon ([`logo.png`](assets/logo.png), 512×512)
- [`README.md`](README.md) / [`README.zh-CN.md`](README.zh-CN.md) — English and Simplified Chinese documentation
- [`CHANGELOG.md`](CHANGELOG.md) — Release history
- [`LICENSE`](LICENSE) — MIT License
