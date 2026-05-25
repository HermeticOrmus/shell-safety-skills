<p align="center">
  <img src="https://ormus.solutions/mascot/pixellab_liquid_to_key.gif" alt="Shell Safety Skills" width="128" style="image-rendering: pixelated;" />
</p>

<h1 align="center">Shell Safety Skills</h1>

<p align="center">
  <em>A CLAUDE.md for shell-safety discipline — set -euo pipefail boilerplate, proper quoting, array usage, error handling, anti-patterns. Universal bash hygiene for AI-generated shell scripts.</em>
</p>

<p align="center">
  <a href="https://github.com/HermeticOrmus/shell-safety-skills/stargazers"><img src="https://img.shields.io/github/stars/HermeticOrmus/shell-safety-skills?style=flat-square&color=aa8142" alt="Stars" /></a>
  <a href="https://github.com/HermeticOrmus/shell-safety-skills/blob/main/LICENSE"><img src="https://img.shields.io/github/license/HermeticOrmus/shell-safety-skills?style=flat-square&color=aa8142" alt="License" /></a>
  <a href="https://github.com/HermeticOrmus/shell-safety-skills/commits"><img src="https://img.shields.io/github/last-commit/HermeticOrmus/shell-safety-skills?style=flat-square&color=aa8142" alt="Last Commit" /></a>
  <img src="https://img.shields.io/badge/Claude_Code-aa8142?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code" />
</p>

---

> **A single `CLAUDE.md` for shell-safety discipline. `set -euo pipefail` boilerplate, proper quoting, array handling, error patterns, anti-patterns. Universal bash hygiene for AI-generated shell scripts.**

## The problem

Bash is forgiving in dangerous ways. Scripts run to completion past failures by default. Unset variables expand to empty strings. Word-splitting happens on spaces inside filenames. AI-generated shell scripts inherit these defaults — which means AI-generated shell is often subtly wrong in ways that pass tests but break in production.

This file installs the discipline that catches common bash bugs at the script's first failure rather than its hundredth side effect.

## The boilerplate

Every script starts with:

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

That's the cost of entry. Three lines.

## What this covers

| Category | Rule |
|---|---|
| Boilerplate | `set -euo pipefail` + `IFS=$'\n\t'` always |
| Quoting | Always quote variable expansions; `"$@"` not `$*` |
| Conditionals | `[[ ... ]]` over `[ ... ]`; `(( ... ))` for numeric |
| Arrays | `args=(...)` then `"${args[@]}"` |
| Error handling | Explicit `if ! cmd` when `set -e` would mask intent |
| Subshells | `done < <(cmd)` not `cmd | while read` |
| Destructive ops | `rm -rf "${var:?}/..."` to refuse on unset |
| Anti-patterns | Missing shebang, unquoted vars, `cd` without check, `eval` |
| Heredocs | `<<'EOF'` for literal, `<<EOF` for interpolated |

Full content: [`CLAUDE.md`](CLAUDE.md). Worked examples: [`EXAMPLES.md`](EXAMPLES.md).

## Install

### As a project CLAUDE.md

Drop [`CLAUDE.md`](CLAUDE.md) at the root of your repository (or alongside a `scripts/` directory).

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/HermeticOrmus/shell-safety-skills/main/CLAUDE.md
```

### As a Claude Code skill

The same content as an installable skill: [`skills/shell-safety/`](skills/shell-safety/).

### In Cursor

See [`CURSOR.md`](CURSOR.md). Rule at [`.cursor/rules/shell-safety.mdc`](.cursor/rules/shell-safety.mdc).

## Why this exists

I've shipped enough bash to know the failure modes. The same patterns recur:

- A script "succeeds" because `set -e` wasn't set and the first command silently failed
- `rm -rf "$dir/"` ran on an unset `$dir` and removed `/`
- A `while read | ...` loop populated a variable that vanished after the pipe
- `eval "$untrusted"` allowed command injection
- A heredoc that should have been literal expanded `$secret` into the output

These aren't subtle. They're well-known. They keep happening because the language's defaults invite them. This `CLAUDE.md` is the discipline that overrides the defaults.

## When to skip the boilerplate

For genuinely one-line scripts with no input, no destructive ops, and no chance of partial failure:

```bash
#!/usr/bin/env bash
echo "$USER"
```

Use judgment. The discipline is for scripts that *do something*.

## See also

- [`andrej-karpathy-skills`](https://github.com/HermeticOrmus/andrej-karpathy-skills) — Karpathy's general coding discipline
- [`vibe-engineer-skills`](https://github.com/HermeticOrmus/vibe-engineer-skills) — how to direct AI codegen well
- [`markdown-discipline-skills`](https://github.com/HermeticOrmus/markdown-discipline-skills) — discipline for AI-generated markdown
- [Bash Pitfalls](https://mywiki.wooledge.org/BashPitfalls) — the canonical longer reference
- [ShellCheck](https://www.shellcheck.net/) — static analyzer that catches most of what this file warns about; run it in CI

## Contributing

PRs welcome for additional anti-patterns, real-world failure examples, and adaptations for `zsh` / `fish` where the patterns diverge.

## License

MIT.


---

## Part of the Libre Open-Source Stack for Claude Code

This repository is part of a growing family of open-source toolkits for Claude Code.

### Libre suite — comprehensive plugin bundles

- [LibreUIUX-Claude-Code](https://github.com/HermeticOrmus/LibreUIUX-Claude-Code) — UI/UX development (152 agents, 70 plugins, 76 commands, 74 skills)
- [LibreArch-Claude-Code](https://github.com/HermeticOrmus/LibreArch-Claude-Code) — Software architecture and system design
- [LibreCopy-Claude-Code](https://github.com/HermeticOrmus/LibreCopy-Claude-Code) — Technical writing and documentation engineering
- [LibreDevOps-Claude-Code](https://github.com/HermeticOrmus/LibreDevOps-Claude-Code) — DevOps engineering and infrastructure automation
- [LibreEmbed-Claude-Code](https://github.com/HermeticOrmus/LibreEmbed-Claude-Code) — Embedded systems, firmware, and IoT development
- [LibreFinTech-Claude-Code](https://github.com/HermeticOrmus/LibreFinTech-Claude-Code) — Financial technology development
- [LibreGEO-Claude-Code](https://github.com/HermeticOrmus/LibreGEO-Claude-Code) — AI-search optimization (ChatGPT, Perplexity, Gemini, Google AI Overviews)
- [LibreGameDev-Claude-Code](https://github.com/HermeticOrmus/LibreGameDev-Claude-Code) — Game development across Godot, Unity, Unreal
- [LibreMLOps-Claude-Code](https://github.com/HermeticOrmus/LibreMLOps-Claude-Code) — ML engineering and AI operations
- [LibreMobileDev-Claude-Code](https://github.com/HermeticOrmus/LibreMobileDev-Claude-Code) — Mobile app development (Flutter, React Native, native iOS, native Android)
- [LibreSecOps-Claude-Code](https://github.com/HermeticOrmus/LibreSecOps-Claude-Code) — Security operations

### Skills mini-repos — single CLAUDE.md drop-ins

- [vibe-engineer-skills](https://github.com/HermeticOrmus/vibe-engineer-skills) — Direct AI codegen well (hypothesis → scope → validate → reject working-but-wrong)
- [markdown-discipline-skills](https://github.com/HermeticOrmus/markdown-discipline-skills) — Strip AI-slop from markdown (no em dashes, no marketing fluff)
- [commit-standard-skills](https://github.com/HermeticOrmus/commit-standard-skills) — Ormus Commit Standard v1.0 + commit-msg hook + commitlint
- [unwoke-skills](https://github.com/HermeticOrmus/unwoke-skills) — Strip AI theater (ten sins to eliminate, symmetric engagement)
- [python-conventions-skills](https://github.com/HermeticOrmus/python-conventions-skills) — Modern Python 3.11+ (types, pathlib, async, ruff, mypy, uv)
- [typescript-conventions-skills](https://github.com/HermeticOrmus/typescript-conventions-skills) — TypeScript strict mode, discriminated unions, Result types
- [hermetic-laws-skills](https://github.com/HermeticOrmus/hermetic-laws-skills) — Seven Hermetic Principles applied to engineering
- [riper-workflow-skills](https://github.com/HermeticOrmus/riper-workflow-skills) — Research / Innovate / Plan / Execute / Review systematic dev
- [six-day-cycle-skills](https://github.com/HermeticOrmus/six-day-cycle-skills) — Sustainable shipping cadence with mandatory rest
- [token-optimization-skills](https://github.com/HermeticOrmus/token-optimization-skills) — Claude Code token + context optimization
- [osint-skills](https://github.com/HermeticOrmus/osint-skills) — OSINT research methodology (multi-wave investigative spiral)
- [calcinate-skills](https://github.com/HermeticOrmus/calcinate-skills) — Stage 1 of the Magnum Opus (burn project bloat)
- [claude-md-overhaul-skills](https://github.com/HermeticOrmus/claude-md-overhaul-skills) — Audit CLAUDE.md and MEMORY.md against caps
- [session-handoff-skills](https://github.com/HermeticOrmus/session-handoff-skills) — Session handoff + pickup discipline
- [naming-skills](https://github.com/HermeticOrmus/naming-skills) — Product naming methodology (mine the brand's vocabulary)
- [magnum-opus-skills](https://github.com/HermeticOrmus/magnum-opus-skills) — Seven-stage alchemy applied to project transformation

### Template source

- [andrej-karpathy-skills](https://github.com/HermeticOrmus/andrej-karpathy-skills) — the canonical single-file CLAUDE.md pattern (fork of jiayuan_jy's original)

Star the family, not just one — that's how the suite stays coherent.
