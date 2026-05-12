# Bandit Level 10 → Level 11

## Goal

The password for the next level is stored in the file `data.txt`, which contains base64 encoded data.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit10`
- **Password:** the password obtained from Level 9 → 10

---

## Hint

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

### How to get items that appear only once

The `-u` option of `uniq` prints only lines that are not repeated. When combined with `sort`, this gives you items that appear exactly once in the file.

```bash
sort [file name] | uniq -u
```

### How to count occurrences of each line

The `-c` option of `uniq` prepends each line with the number of times it occurred.

```bash
sort [file name] | uniq -c
```

### How to print human-readable data

The `strings` command extracts printable text from binary files.

```bash
strings [file name]
```

### How to identify Base64 data

Base64 is an **encoding** (not encryption) that represents binary data as text. You can recognize Base64 data by these traits:

- Uses only **64 characters + padding**: `A–Z`, `a–z`, `0–9`, `+`, `/`, and `=` for padding.
- No spaces, no other special characters.
- Length is always a **multiple of 4** (padded with `=` or `==` at the end if needed).
- Output length is roughly **1.33× the original size** (every 3 bytes become 4 characters).

Example of a Base64-encoded string:

```
VGhlIHBhc3N3b3JkIGlzIDEyMzQ1==
```

Quick check on a file:

```bash
file data.txt          # often shows "ASCII text"
cat data.txt           # if you only see A–Z, a–z, 0–9, +, /, =, it's likely Base64
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
ssh -p 2220 bandit10@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 9 → 10.

### 2. List the files in the current directory

```bash
ls
```

After running `ls`, you will see a single file named `data.txt`.

### 3. Check data.txt's contents

```bash
file data.txt
cat data.txt
```

`file` reports the file as ASCII text. Looking at `cat data.txt`, the content consists only of letters, digits, `+`, `/`, and ends with `=`. This matches the Base64 character set, so the file is Base64-encoded data.

Example of what the content looks like:

```
VGhlIHBhc3N3b3JkIGlzIDxQQVNTV09SRD4K
```

### 4. Decode the Base64 data

Use the `base64 -d` command to decode the file:

```bash
base64 -d data.txt
```

**How this command works:**

- `base64` — the encoding/decoding tool.
- `-d` — decode mode (without `-d`, the command would encode instead).
- `data.txt` — the file containing the Base64-encoded content.

The output will reveal a line containing the password for `bandit11`.

---

## Reference

- [Bandit Level 11 Page](https://overthewire.org/wargames/bandit/bandit11.html)