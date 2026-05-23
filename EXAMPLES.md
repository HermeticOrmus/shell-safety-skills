# Examples

Real failures and how the discipline catches them.

---

## 1. Missing `set -e`

**Before**:
```bash
#!/usr/bin/env bash
mkdir /tmp/build
cp -r src/ /tmp/build/
make -C /tmp/build all
deploy /tmp/build/output
```

**Failure mode**: `cp` fails (disk full, permission, source missing). Script continues. `make` runs on incomplete tree. `deploy` ships broken artifact.

**After**:
```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
mkdir /tmp/build
cp -r src/ /tmp/build/
make -C /tmp/build all
deploy /tmp/build/output
```

Now `cp` failing exits the script before `make` runs.

---

## 2. Unset variable explosion

**Before**:
```bash
#!/usr/bin/env bash
BACKUP_DIR=/var/backups/$ENV
rm -rf "$BACKUP_DIR/old/*"
```

**Failure mode**: `$ENV` is unset. `$BACKUP_DIR` becomes `/var/backups/`. `rm -rf` targets `/var/backups//old/*`. Could be worse with different paths.

**After**:
```bash
#!/usr/bin/env bash
set -euo pipefail
BACKUP_DIR="/var/backups/${ENV:?}"
rm -rf "${BACKUP_DIR:?}/old/"*
```

`${ENV:?}` exits with error if unset. `${BACKUP_DIR:?}` does the same. The script fails loudly instead of silently nuking.

---

## 3. Unquoted variable in path

**Before**:
```bash
FILE=$1
cat $FILE
```

**Failure mode**: `$1` is `report 2026.txt`. `cat` receives `report` and `2026.txt` as two arguments. One file doesn't exist; the other gets opened. Or worse, `$1` is `$(rm -rf /)` (theoretical, but the principle applies to all interpolation).

**After**:
```bash
FILE="$1"
cat "$FILE"
```

Quotes preserve word boundaries.

---

## 4. `cd` without check

**Before**:
```bash
#!/usr/bin/env bash
cd /etc/myapp
rm -rf *.bak
```

**Failure mode**: `/etc/myapp` doesn't exist. `cd` prints error to stderr but the script continues from the current directory (which might be `/`). `rm -rf *.bak` runs in the wrong directory.

**After**:
```bash
#!/usr/bin/env bash
set -euo pipefail
cd /etc/myapp
rm -rf -- *.bak
```

With `set -e`, the failing `cd` exits the script. The `--` after `rm -rf` prevents filenames starting with `-` from being interpreted as flags.

---

## 5. Pipeline silently failing

**Before**:
```bash
grep ERROR app.log | sort | uniq -c
```

**Failure mode**: `app.log` doesn't exist. `grep` fails. The pipeline's exit status is `0` (from `uniq`), so no error is reported. The output is empty; the caller can't distinguish "no errors" from "log file missing."

**After**:
```bash
set -o pipefail
grep ERROR app.log | sort | uniq -c
```

With `pipefail`, the pipeline returns the first failing command's status. Combined with `set -e`, the script exits.

---

## 6. Word-splitting on filenames

**Before**:
```bash
for f in $(ls *.log); do
    grep ERROR "$f"
done
```

**Failure mode**: A file named `my error.log` becomes two iterations: `my` and `error.log`. Both fail to grep.

**After**:
```bash
for f in *.log; do
    grep ERROR "$f"
done
```

Use glob directly. No `ls` parsing. No word-splitting on filenames.

---

## 7. `while read | ...` loop variable scoping

**Before**:
```bash
COUNT=0
find . -name "*.txt" | while read -r file; do
    COUNT=$((COUNT + 1))
done
echo "Found $COUNT files"
# Always prints "Found 0 files"
```

**Failure mode**: The `while` loop runs in a subshell because of the pipe. `COUNT` is incremented in the subshell. When the subshell exits, those changes are lost. The outer `COUNT` is still 0.

**After**:
```bash
COUNT=0
while read -r file; do
    COUNT=$((COUNT + 1))
done < <(find . -name "*.txt")
echo "Found $COUNT files"
# Prints the actual count
```

Process substitution keeps the `while` loop in the current shell.

---

## 8. Array vs. string

**Before**:
```bash
FLAGS="--foo --bar --baz=qux quux"
mycommand $FLAGS
```

**Failure mode**: `--baz=qux quux` is a single value with a space. `mycommand` sees `--foo`, `--bar`, `--baz=qux`, `quux` as four separate arguments. The `quux` argument is mis-attributed.

**After**:
```bash
FLAGS=(--foo --bar --baz="qux quux")
mycommand "${FLAGS[@]}"
```

Array with quoted expansion preserves the `qux quux` as a single argument.

---

## 9. Heredoc with sensitive data

**Before**:
```bash
SECRET="hunter2"
cat <<EOF > /etc/myapp/config.json
{
  "password": "$SECRET"
}
EOF
```

**Failure mode**: The `EOF` here is unquoted, so the heredoc interpolates `$SECRET`. That's intentional. But if you have a literal `$SECRET` placeholder that should NOT be interpolated (because the template gets passed to a downstream tool that does the interpolation), you'll silently substitute the wrong value.

**After (when you want interpolation)**:
```bash
cat <<EOF > /etc/myapp/config.json
{
  "password": "$SECRET"
}
EOF
```

**After (when you want literal `$SECRET`)**:
```bash
cat <<'EOF' > /etc/myapp/config.template
{
  "password": "$SECRET"
}
EOF
```

The single-quoted `'EOF'` makes the heredoc literal.

---

## 10. The `cd` that escaped quoting

**Before**:
```bash
DIR=$user_input
cd $DIR
do_dangerous_thing
```

**Failure mode**: `$user_input` is `; rm -rf ~`. `cd` does nothing useful, then the shell parses `; rm -rf ~` as a separate command. Game over.

**After**:
```bash
DIR="$user_input"
cd "$DIR"
do_dangerous_thing
```

Quotes make the entire interpolation a single argument to `cd`. `cd` fails (no such directory) and with `set -e`, the script exits.

For extra safety on user input: validate the path before using it.

---

## 11. `eval` injection

**Before**:
```bash
read -r COMMAND
eval "$COMMAND"
```

**Failure mode**: User types literally anything. Your script runs it.

**After**:
```bash
read -r COMMAND
# Don't eval. If you need to dispatch, use a case statement:
case "$COMMAND" in
    start) systemctl start myapp ;;
    stop) systemctl stop myapp ;;
    status) systemctl status myapp ;;
    *) echo "Unknown command: $COMMAND" >&2; exit 1 ;;
esac
```

`eval` should be your absolute last resort and even then it's usually wrong.

---

## 12. Reading file content with `for`

**Before**:
```bash
for line in $(cat config.txt); do
    process "$line"
done
```

**Failure mode**: Multiple things go wrong. Each WORD becomes a `$line`, not each line. Lines with spaces split. Empty lines disappear. Lines with glob chars (`*`, `?`) get expanded.

**After**:
```bash
while IFS= read -r line; do
    process "$line"
done < config.txt
```

`IFS=` prevents leading/trailing whitespace stripping. `-r` prevents backslash interpretation. The redirect feeds the loop properly.

---

## 13. `printf` for safe output of untrusted data

**Before**:
```bash
LOG_LINE="-n hello"
echo "$LOG_LINE"
# Prints: hello   (no newline because echo interpreted -n as a flag)
```

**After**:
```bash
LOG_LINE="-n hello"
printf '%s\n' "$LOG_LINE"
# Prints: -n hello
```

`echo` flag parsing on untrusted input is a real problem. `printf '%s\n'` is the safe alternative.

---

## 14. The classic `IFS` trap

**Before**:
```bash
#!/usr/bin/env bash
# IFS not set
files=(*.txt)
for f in ${files[@]}; do
    process "$f"
done
```

**Failure mode**: Unquoted `${files[@]}` undergoes word splitting on default IFS (space, tab, newline). A file named `my report.txt` becomes two arguments.

**After**:
```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
files=(*.txt)
for f in "${files[@]}"; do
    process "$f"
done
```

Quoted `"${files[@]}"` preserves array boundaries. `IFS=$'\n\t'` is belt-and-suspenders.

---

## 15. Capturing exit code without losing it

**Before**:
```bash
do_thing
if [[ $? -ne 0 ]]; then
    cleanup  # cleanup is a command — its exit code now overrides
    echo "do_thing failed"
fi
```

**Failure mode**: Between `do_thing` and the `[[ $? -ne 0 ]]` check, you might inadvertently run another command (the `if` evaluation is fine, but a print or log between would overwrite `$?`).

**After**:
```bash
if ! do_thing; then
    cleanup
    echo "do_thing failed"
fi
```

Don't reach for `$?` when you can use `if !` directly. Cleaner, no race window.

If you need to capture the exit code for later:
```bash
do_thing
rc=$?
if (( rc != 0 )); then
    echo "do_thing failed with $rc"
fi
```

---

## How to apply this

1. **Start every new script with the boilerplate**. Three lines. Not optional.
2. **Quote everything**. Even when you "know" the variable can't have spaces. Make quoting reflexive.
3. **Use arrays for argument lists**. Never build a flag string with spaces.
4. **Run ShellCheck**. It catches most of the above mechanically.
5. **Be skeptical of AI-generated shell**. The AI's defaults match bash's defaults, which are unsafe. Verify every script against this list.

## See also

- [`shellcheck`](https://www.shellcheck.net/) — static analysis that mechanically catches most anti-patterns here
- [`shfmt`](https://github.com/mvdan/sh) — formatter
- [Bash Pitfalls (wooledge wiki)](https://mywiki.wooledge.org/BashPitfalls) — the canonical longer reference, ~50 numbered traps with examples
