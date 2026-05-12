# Bandit Level 6 → Level 7

## Goal

The goal of this level is to get the password for `bandit7`.

The password file is stored **somewhere on the server** and meets all of the following conditions:

- owned by user `bandit7`
- owned by group `bandit6`
- 33 bytes in size

---
## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit6`
- **Password:** the password obtained from Level 5

---
## Hint

- `ls`
- `cd`
- `cat`
- `file`
- `du`
- `find`
- `grep`

---
## Solution

### How to list files in the current directory

```bash
ls
```

### How to change directory

```bash
cd [directory name]
```

### How to read a file's contents

```bash
cat [file name]
```

### How to find files recursively from the root directory

To search the entire filesystem, start `find` from the root directory `/`:

```bash
find / -type f
```

### How to filter files by owner with `find`

The `-user` option matches files owned by a specific user. The `-group` option matches files owned by a specific group:

```bash
find / -user bandit7              # files owned by user bandit7
find / -group bandit6             # files owned by group bandit6
find / -user bandit7 -group bandit6   # both conditions (AND)
```

### How to filter files by size with `find`

The `-size` option filters by file size. The `c` suffix means bytes:

```bash
find / -size 33c                  # files exactly 33 bytes in size
```

### How to suppress error messages with `2>/dev/null`

When searching from `/`, `find` will try to enter directories you do not have permission to access, producing many `Permission denied` errors. These errors are written to **standard error (stderr)**, not standard output. We can redirect stderr to `/dev/null` (a special file that discards anything written to it) to hide them:

```bash
find / 2>/dev/null
```

| Stream | Number | Purpose                          |
| ------ | ------ | -------------------------------- |
| stdin  | 0      | Standard input                   |
| stdout | 1      | Standard output (normal results) |
| stderr | 2      | Standard error (error messages)  |

`2>/dev/null` means "redirect stream 2 (stderr) to `/dev/null`".

---
## Steps

### 1. Log in with SSH

```bash
ssh -p 2220 bandit6@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 5.

### 2. Understand the search scope

Unlike previous levels, the password file is **not** in your home directory. It is stored somewhere on the entire filesystem, so we need to search from the root directory `/`.

### 3. Find the password file by combining all three conditions

The problem gives us three conditions: owned by user `bandit7`, owned by group `bandit6`, and exactly 33 bytes in size. We can combine all three in a single `find` command:

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

**How this command works:**

- `find /` — start the search from the root directory.
- `-user bandit7` — match only files owned by user `bandit7`.
- `-group bandit6` — match only files owned by group `bandit6`.
- `-size 33c` — match only files exactly 33 bytes in size.
- `2>/dev/null` — discard `Permission denied` and other error messages so the output stays clean.

The output will look something like this:

```
/var/lib/dpkg/info/bandit7.password
```

### 4. Read the identified file

```bash
cat /var/lib/dpkg/info/bandit7.password
```

The output will display the password needed to log in as `bandit7` in the next level.

## Deep Dive: Standard streams and redirection

This level introduces a powerful concept: **redirecting output streams**. Every Linux process has three default streams:

| Stream | Number | Description                                               |
| ------ | ------ | --------------------------------------------------------- |
| stdin  | 0      | Where the program reads input from (usually the keyboard) |
| stdout | 1      | Where normal output goes (usually the terminal)           |
| stderr | 2      | Where error messages go (also usually the terminal)       |

### Why are stdout and stderr separate?

Although both stdout and stderr appear on the terminal by default, they are **two different streams**. This separation lets you handle results and errors independently. For example, when running `find /`, useful results and `Permission denied` errors are interleaved on screen, but they actually travel through different channels.

### Redirection operators

| Operator      | Meaning                                     |
| ------------- | ------------------------------------------- |
| `>` or `1>`   | Redirect stdout to a file (overwrite)       |
| `>>` or `1>>` | Redirect stdout to a file (append)          |
| `2>`          | Redirect stderr to a file                   |
| `2>>`         | Redirect stderr to a file (append)          |
| `&>`          | Redirect both stdout and stderr             |
| `2>&1`        | Redirect stderr to wherever stdout is going |

### What is `/dev/null`?

`/dev/null` is a special file that **discards everything written to it**. It is often called the "black hole" of Linux. Reading from it returns nothing (EOF), and writing to it succeeds but the data vanishes.

So `2>/dev/null` literally means: "send all error messages into the black hole."

### Practical examples

```bash
find / -name "*.conf" 2>/dev/null              # hide permission errors
find / 2>errors.log                            # save errors to a file
find / >results.txt 2>/dev/null                # results to file, errors discarded
find / >/dev/null 2>&1                         # discard everything
find / -name "*.txt" 2>&1 | grep -i "config"   # merge stderr into stdout, then grep
```

Without `2>/dev/null`, the result of this level would be buried under thousands of `Permission denied` lines, making the actual password file hard to spot.

---
## Reference

- [Bandit Level 7 Page](https://overthewire.org/wargames/bandit/bandit7.html)