# CLAUDE.md

Shell-safety discipline. Apply to every shell script written or modified. Bash is forgiving in dangerous ways — these defaults catch the common bugs early.

**Scope**: `**/*.sh`, `**/*.bash`, `**/Makefile`, `**/.bashrc`, `**/.zshrc`, `**/scripts/**`.

**Tradeoff**: bias toward strict over permissive. For one-line scripts with no inputs and no consequences, use judgment.

## Boilerplate every script needs

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

- `set -e` — exit on any non-zero return
- `set -u` — exit on use of unset variable
- `set -o pipefail` — pipeline returns the failing command's status, not just the last
- `IFS=$'\n\t'` — splits on newlines + tabs only, not spaces (catches "filename with spaces" bugs)

Without these, bash will happily continue past failures, expand unset variables to empty strings, and split words on spaces inside filenames. All three are common bug sources.

## Quoting

- **Always quote variable expansions**: `"$var"`, `"${var}"`, `"$@"`
- Unquoted `$var` is word-splitting waiting to happen
- `"$@"` for forwarding all positional args (NOT `$*`, NOT unquoted `$@`)

## Conditionals

- `[[ ... ]]` over `[ ... ]` (no word-splitting inside, supports `&&` / `||` / regex)
- File tests: `[[ -f file ]]`, `[[ -d dir ]]`, `[[ -r file ]]`
- String comparison: `[[ "$a" == "$b" ]]` (quotes inside `[[` are still safer)
- Numeric comparison: `(( a > b ))`

## Arrays

- Use arrays for lists: `args=(--foo bar --baz "qux quux")`
- Expand with `"${args[@]}"` for proper word handling
- Never pass `"${args[*]}"` to a command unless you want them all in one string

## Error handling

- Check command success explicitly when `set -e` would mask intent: `if ! cmd; then ...`
- For optional/expected failures: `cmd || true`
- Capture exit code when needed: `cmd; rc=$?` then `if (( rc == 0 )); then ...`

## Subshells and process substitution

- Prefer process substitution `<(cmd)` over temp files for one-shot pipelines
- Watch for `while read | ...` — the loop runs in a subshell so variable changes don't escape; use `done < <(cmd)` instead

## Destructive ops

- `rm -rf "$var/"` without verifying `$var` is non-empty is a footgun. Use the trailing-slash trick to make bash refuse on unset:

```bash
rm -rf "${var:?}/..."
```

The `${var:?}` syntax errors if `var` is unset or empty, preventing the catastrophic `rm -rf /...`.

## Anti-patterns

- No shebang or wrong shebang (`#!/bin/sh` when bash features are used)
- Missing `set -euo pipefail`
- Unquoted variable expansions
- `cd` without checking it succeeded:
  ```bash
  cd /some/dir && do-stuff   # safe
  cd /some/dir; do-stuff     # do-stuff runs from wherever, even on cd failure
  cd /some/dir || exit       # safe alternative
  ```
- `rm -rf "$var/"` without verifying `$var` is non-empty
- Using `eval` (almost always there's a safer way; if you must, sanitize aggressively)
- Long pipelines without `pipefail` (silent failures in the middle)
- Heredoc without proper quoting:
  ```bash
  # Variables expanded in heredoc
  cat <<EOF
  $variable
  EOF

  # Variables NOT expanded — use this for literal content
  cat <<'EOF'
  $variable
  EOF
  ```

## When the script needs root

- Sudo at the specific command, not the whole script. Don't `sudo bash script.sh` if you can `script.sh` with `sudo` on the 3 lines that need it.
- For installers: document what each `sudo` call is doing. Future-you will thank present-you.

## When the script processes external input

- Validate at the boundary. Don't trust filenames, env vars, or stdin to be well-formed.
- Use `printf '%s\n' "$untrusted"` not `echo "$untrusted"` if `$untrusted` might start with `-`.
- Quote everything that goes into commands. Variable interpolation into shell strings is injection-prone.

---

**License**: MIT — use it, fork it, merge it into your own CLAUDE.md.
