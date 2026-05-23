# Using this repo with Cursor

This project includes a **Cursor project rule** so the shell-safety discipline applies automatically.

## In this repository

1. Open the folder in Cursor.
2. The rule [`.cursor/rules/shell-safety.mdc`](.cursor/rules/shell-safety.mdc) is committed with the appropriate scope.
3. Confirm under **Settings → Rules**.

## Use the same discipline in another project

**Cursor**: Copy `.cursor/rules/shell-safety.mdc` into that project's `.cursor/rules/` directory.

**Other AI tools**: Copy [`CLAUDE.md`](CLAUDE.md) to the project root.

## Optional: personal skill

The same content as a reusable skill at [`skills/shell-safety/SKILL.md`](skills/shell-safety/SKILL.md). Copy or symlink into your personal skills directory.

## ShellCheck integration

The discipline pairs well with [ShellCheck](https://www.shellcheck.net/) for mechanical enforcement. Add it to your CI:

```yaml
- uses: ludeeus/action-shellcheck@master
```
