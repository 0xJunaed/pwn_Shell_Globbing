
### Concept
Glob = shell feature. When an argument has a wildcard, the **shell** (not the command) replaces it with all matching filenames *before* running the command.

| Glob | Matches |
|------|---------|
| `*` | zero or more characters |
| `?` | exactly one character (any) |
| `[abc]` | exactly one character from the set |

Common rules for all globs:
- Never match `/`
- Never match a leading `.` (hidden files)
- No match = glob stays as **literal text** (default behavior)

---

### 1. `*` Wildcard

```bash
touch file_a file_b file_c
ls
# file_a  file_b  file_c

echo Look: file_*
# Look: file_a file_b file_c
```
`file_*` expanded into 3 separate arguments.

**Matches only one file:**
```bash
touch file_a
echo Look: file_*
# Look: file_a
```

**Zero matches:**
```bash
echo Look: nope_*
# Look: nope_*
```
! Nothing matched, so the shell leaves the glob unchanged.

**Using `*` in paths:**
```bash
echo ONE: /ho*/*ck*
# ONE: /home/hacker

echo TWO: /*/hacker
# TWO: /home/hacker

echo THREE: ../*
# THREE: ../hacker
```
- `/ho*/*ck*` → `/home/hacker`
- `/*/hacker` → `*` fills one directory level (`home`)
- `../*` → everything inside the parent directory


### 2. `?` Wildcard

`?` = matches **exactly one character**. Like `*`, but only one.

```bash
touch file_a file_b file_cc
ls
# file_a  file_b  file_cc

echo Look: file_?
# Look: file_a file_b

echo Look: file_??
# Look: file_cc
```
`file_?` = `file_` + exactly 1 character, so `file_cc` is not matched. `??` = exactly 2 characters.

##### My Finding (Challenge)
* Task: `cd /challenge` without typing `c`, `l` or `*`.

**Attempt 1: failed**
```bash
cd /cha??enge
# You used either the 'c', 'l', or '*' characters. Disallowed!
```
- The glob was fine, but the typed letter `c` in `cha` was blocked.

**Attempt 2: worked**
```bash
cd /?ha??????
./run
# prints the flag
```
- `challenge` has 9 letters:
	- `?` = `c`
	- `h` `a` = typed as is
	- `??????` = `l l e n g e` (6 chars)

 Count the characters. The number of `?` must match the exact length.
 
### 3. `[]` Wildcard

`[]` = matches **exactly one character**, but only from the set inside the brackets. A limited form of `?`.

`[pwn]` matches `p`, `w`, or `n`.

```bash
touch file_a file_b file_c
echo Look: file_[ab]
# Look: file_a file_b
```
`file_[ab]` = `file_` + `a` or `b`, so `file_c` is not matched.

##### Globbing Works on Paths
Globbing happens on a **path basis**, so you can expand entire paths, not just filenames in the current directory.

```bash
echo Look: /home/hacker/file_[ab]
# Look: /home/hacker/file_a /home/hacker/file_b
```
The path prefix stays, only the glob part expands. Each result is a full absolute path.

##### My Finding 1 (Challenge: Same Directory)
Task: in `/challenge/files`, run `/challenge/run` with **one argument** that bracket-globs into `file_b`, `file_a`, `file_s`, `file_h`.

Files in the folder: `file_a` to `file_z`.

**Solution:**
```bash
cd /challenge/files
/challenge/run file_[absh]
# You got it! Here is your flag!
```
`file_[absh]` expands into `file_a file_b file_h file_s` (4 arguments from 1 glob).

**Mistakes I made:**
- `/.run file_[absh]` failed: `No such file or directory`. I typed `/.run` instead of `./run`. The `.` and `/` order matters.
- `./run file_[absh]` from `/challenge` failed: `please run with a working directory of /challenge/files!`. I had to `cd` into `/challenge/files` first and use the full path `/challenge/run`.

- The working directory decides which files the glob matches. Wrong directory = wrong expansion.

##### My Finding 2 (Challenge: From Home Directory)
Task: starting from home (`~`), run `/challenge/run` with one argument that globs into the **absolute paths** of `file_b`, `file_a`, `file_s`, `file_h`.

**Attempt 1: failed**
```bash
/challenge/run file_[absh]
# Error: you will need to specify the path to the files as part of your glob
# argument, since they are in a different directory than your current working directory!
```
 No path in the glob = shell looks only in the current directory (`~`), finds nothing, glob stays literal.

**Attempt 2: worked**
```bash
/challenge/run /challenge/files/file_[absh]
# You got it! Here is your flag!
```
Expands into:
`/challenge/files/file_a /challenge/files/file_b /challenge/files/file_h /challenge/files/file_s`

**Mistake I made:**
- I ran `/challenge/files/file_[absh]` alone (without `/challenge/run` in front). The shell tried to run the first expanded path as a command. The glob must be an **argument** to a command.

**Test with echo:**
```bash
echo look: challenge/files/file_[absh]
# look: challenge/files/file_[absh]
```
-  No leading `/`, so it is a **relative** path. From `~` there is no `challenge/files` folder, so no match and the glob stays literal. Correct: `/challenge/files/file_[absh]`.

### 4. Mixing Globs (`[]` + `*`)

Globs can be **combined in one argument**. `[]` picks one character from a set, `*` fills in the rest.

##### Challenge
Task: in `/challenge/files`, write a glob of **6 characters or less** that matches **only** `challenging`, `educational`, `pwning`.

**Solution:**
```bash
cd /challenge/files
/challenge/run [cep]*
# You got it! Here is your flag!
```

##### How I Thought About It
The names look totally different, but the **first letters** are the pattern:
- `challenging` starts with **c**
- `educational` starts with **e**
- `pwning` starts with **p**

So:
- `[cep]` = first character must be `c`, `e` or `p`
- `*` = the rest of the name can be anything

`[cep]*` is exactly 6 characters, which fits the limit.

##### Tricky Things
- Different-looking names can share a pattern. Look at the **start, end or middle** of the names for something in common.

	 `[cep]*` matches **any** file starting with `c`, `e` or `p`. It only worked because no other file in the folder starts with those letters. Always check the folder first.

	 The 6-character limit is the point: you cannot type the full names. Let `[]` handle the unique part and `*` handle the long part.

#### Other Ways to Mix (Examples)
```bash
echo file_[ab]*                # starts with file_a or file_b, anything after
echo *[0-9]                    # anything ending with a digit
echo ?at*                      # one any char + "at" + anything
echo /challenge/files/[cep]*   # works with full paths too
```


### 5. Exclusionary Globbing (`[!...]`)

`[!...]` = matches **exactly one character that is NOT in the set**. The opposite of `[]`.

`[!pwn]` matches any single character **except** `p`, `w`, or `n`.

```bash
touch file_a file_b file_c
echo Look: file_[!ab]
# Look: file_c
```
`file_[!ab]` = `file_` + any one character that is not `a` or `b`, so only `file_c` matches.

#### With `*` (like `[!pwn]*`)
```bash
echo [!pwn]*
```
- files whose **first character is not** `p`, `w` or `n`, and anything after it.

Example in a folder with `pwning`, `challenging`, `educational`:
```bash
echo [!pwn]*
# challenging educational
```
`pwning` is excluded because it starts with `p`.

##### Compare
| Glob | Matches |
|------|---------|
| `[cep]*` | starts with `c`, `e` or `p` |
| `[!cep]*` | starts with anything **except** `c`, `e`, `p` |

#### Tricky Things
! The `!` must be the **first character** right after `[`. `[!pwn]` = NOT p/w/n, but `[p!wn]` just means `p`, `!`, `w` or `n`.

! `[!pwn]` still matches **exactly one** character. It does not mean "zero characters" or "any number". Add `*` if you want the rest of the name.

! It is useful when it is easier to say what you **don't** want than what you do want.

! Same rules as other globs: no match = glob stays literal text, hidden files (leading `.`) are skipped.

#### Key Points
- `[!abc]` = one character, **not** a, b or c
- Some shells also accept `[^abc]`, but `[!abc]` is the standard one
- Combine with `*` and `?` like any other glob
- Preview with `echo` or `ls` before running the real command


## Key Points (All Together)
- Expansion is done by the **shell**, before the command runs
- `*` = zero or more characters, within one path level
- `?` = exactly one character, count must match the length
- `[]` = exactly one character from the set, order inside does not matter (`[absh]` = `[hsba]`)
- Expansion output is sorted alphabetically
- Mix `*`, `?` and `[]` in one argument
- Globs work with full paths: `/path/to/file_[ab]`
- Not in the same directory? Put the path **inside** the glob argument
- Leading `/` = absolute path, no `/` = relative to current directory
- Blocked letter? Replace it with `?` or put it inside `[]`
- Look for patterns in names (first letter, last letter, length)
- Use `echo` or `ls` first to preview what a glob expands to
- Make sure the glob matches **only** the files you want
- No match = glob stays as literal text
- Hidden files (starting with `.`) are skipped

