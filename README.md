# Shell Safety Skills

> A single `CLAUDE.md` for shell-safety discipline. `set -euo pipefail` boilerplate, proper quoting, array handling, error patterns, anti-patterns. Universal bash hygiene for AI-generated shell scripts.

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
