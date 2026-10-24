# Shell Globbing Notes

Short notes on shell globbing from pwn.college. Only the commands and tricky parts that most people miss are included.

## What's Inside
- `*` : zero or more characters
- `?` : exactly one character
- `[abc]` : one character from a set
- `[!abc]` : one character **not** in the set
- Globbing on full paths
- Mixing globs (e.g. `[cep]*`)

## Quick Reference
| Glob | Matches |
|------|---------|
| `*` | zero or more characters |
| `?` | exactly one character |
| `[abc]` | one of `a`, `b`, `c` |
| `[!abc]` | any one character except `a`, `b`, `c` |

## Things Most People Don't Know
- The **shell** expands the glob, not the command, and it happens *before* the command runs
- No match = the glob stays as **literal text**
- `*`, `?` and `[]` never match `/` or a leading `.` (hidden files)
- Globs work on **paths**: `/challenge/files/file_[absh]`
- Blocked a letter? Replace it with `?` (e.g. `cd /?ha??????`)
- `[cep]*` can match files with totally different names by using their first letter
- `[!pwn]*` is useful when it is easier to say what you don't want
- Use `echo` first to preview what a glob expands to

## Example
```bash
cd /challenge/files
/challenge/run [cep]*
```

## Files
- `globbing.md` : full notes with examples, mistakes and findings

## Source
[pwn.college](https://pwn.college) - Globbing module
