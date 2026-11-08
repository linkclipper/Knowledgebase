# Research — Windows/PowerShell pitfalls (things that actually bit us)

**Tested:** 2026-10/11 incidents in linkclipper/linkclipper. Keep appending; every entry here cost real debugging time.

## 1. Encoding: the cp1252 rename incident (2026-11)

What happened: a bulk text-rename pass over the extension + site was done with PowerShell `Get-Content`/`Set-Content`. Default `Get-Content` decodes as **cp1252** and `Set-Content`/`Out-File` default to the system ANSI codepage — UTF-8 files got decoded as cp1252 then written as UTF-8, so every multibyte character (— · í etc.) became mojibake (`â€"`, `Ã­`).

Rules:
- Read/write with `-Encoding UTF8` **every time** bulk text goes through PowerShell.
- After any bulk text pass, run the artifact check: search the tree for `/Â|â€/` (both bytes-soup signatures). Zero matches, or show me the diff.
- Prefer doing text transforms in Node (write+run a `.cjs`): `fs.readFileSync(path, 'utf8')` / `writeFileSync(..., 'utf8')` cannot lie.
- PowerShell `-replace` is **case-insensitive**. For case-sensitive regex, use `-creplace`.

## 2. Native command exit codes lie when stderr is redirected

`2>&1` in PowerShell wraps native stderr in `ErrorRecord`s and can flip the script's `$?`/exit code even when the native command succeeded (and masks failure when it didn't). Consequences:
- Don't gate on script exit when stderr noise is expected; gate on **artifacts and explicit asserts**.
- `git push` prints human-visible "main -> main" on stderr — success is proven by the ref, not the exit code.
- Sequences like `SomeNative.exe ... | Out-Null; if ($LASTEXITCODE -ne 0)` are the reliable pattern; the trailing `| Select-Object` after `2>&1` also mangles output order.

## 3. Git annotated tags need an identity

`git tag -a v0.3.0 -m "..."` on a machine without repo-level `user.name` fails **after** earlier commits using `-c user.name=...` succeeded. If `-c` commits were used in this session, same discipline for tags: `git -c user.name=... -c user.email=... tag -a vX -m ...`.
Also: `2>$null` on a failing tag creation eats the error — suppressed failures are how the tag ended up "missing but pushed", falling out of releases.

## 4. Repo creation order that worked

`git init -b main` locally → seed files → commit → `gh repo create linkclipper/<name> --public --source . --push` → verify default branch with `gh repo view` (report says main when the only pushed branch is main — do not assume; check). Creating the repo first over the web and then cloning adds an origin whose default is `main` but no local tracking — the `--source . --push` path keeps local branch → default mapping explicit and avoids the empty-repo clone dance.
