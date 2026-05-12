# Bandit Level 7 → Level 8

## Goal

The goal of this level is to get the password for `bandit8`. The password for the next level is stored in the file `data.txt` next to the word **millionth**.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit7`
- **Password:** the password obtained from Level 6

---

## Hint

- `man`
- `grep`
- `sort`
- `uniq`
- `strings`
- `base64`
- `tr`
- `tar`
- `gzip`
- `bzip2`
- `xxd`

---

## Solution

### How to get command manual

Command `man` prints command's manual.

```bash
man [command]
```

### How to filter lines that contain a pattern

The `grep` command prints lines from its input that match a given pattern:

```bash
grep "pattern" [file name]
```

It is often combined with other commands using a pipe (`|`):

```bash
[command] | grep "pattern"
```

### How to arrange contents of a file

The `sort` command sorts lines of a file alphabetically (or numerically with `-n`).

```bash
sort [file name]
```

### How to get unique data from contents of a file

The `uniq` command removes consecutive duplicate lines. It's usually combined with `sort` since `uniq` only detects duplicates that are adjacent.

```bash
sort [file name] | uniq
```

### How to print human-readable data

The `strings` command extracts printable text from binary files.

```bash
strings [file name]
```

### How to decode Base64 data

The `base64` command encodes or decodes Base64-formatted text.

```bash
base64 -d [file name]      # decode
base64 [file name]         # encode
```

### How to translate or delete characters

The `tr` command translates (replaces) or deletes characters from standard input.

```bash
[command] | tr 'A-Z' 'a-z'     # convert uppercase to lowercase
[command] | tr -d ' '          # delete spaces
```

### How to handle tar archives

The `tar` command bundles or extracts archive files.

```bash
tar -xf [file name]            # extract
tar -cf archive.tar [files]    # create
```

### How to handle gzip / bzip2 compression

`gzip` and `bzip2` are compression utilities. Use the matching `gunzip` / `bunzip2` to decompress.

```bash
gunzip [file.gz]
bunzip2 [file.bz2]
```

### How to handle hex dumps

The `xxd` command creates a hex dump or reverses one back into binary.

```bash
xxd [file name]            # create hex dump
xxd -r [file name]         # reverse hex dump back to binary
```

---

## Steps

### 1. Log in with SSH

```bash
ssh -p 2220 bandit7@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 6 → 7.

### 2. Search for the keyword in data.txt

The file `data.txt` contains thousands of lines. Since the password is on the line next to the word `millionth`, use `grep` to extract it directly.

```bash
grep "millionth" data.txt
```

**How this command works:**

- `grep` — filter lines by pattern.
- `"millionth"` — the keyword to search for.
- `data.txt` — the file to search in.

The output will look something like this:

```
millionth	<PASSWORD_STRING>
```

The string after `millionth` is the password for `bandit8`.

---

## Reference

- [Bandit Level 8 Page](https://overthewire.org/wargames/bandit/bandit8.html)