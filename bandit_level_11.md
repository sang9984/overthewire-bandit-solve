# Bandit Level 11 → Level 12

## Goal

The password for the next level is stored in the file `data.txt`, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit11`
- **Password:** the password obtained from Level 10

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

### How to decode Base64 data

The `base64` command encodes or decodes Base64-formatted text.

```bash
base64 -d [file name]      # decode
base64 [file name]         # encode
```

### What is ROT13?

**ROT13** ("rotate by 13") is a simple letter substitution cipher that shifts every letter **13 positions forward** in the alphabet.

- `A` → `N`, `B` → `O`, ..., `M` → `Z`
- `N` → `A`, `O` → `B`, ..., `Z` → `M`
- Lowercase letters work the same way: `a` → `n`, ..., `z` → `m`
- Digits, spaces, and symbols are **not affected**.

Because the alphabet has 26 letters, applying ROT13 **twice** gives back the original text. This makes encoding and decoding the **same operation** — there's no separate "decode" command.

Example:

```
Plaintext:   Hello World
ROT13:       Uryyb Jbeyq
ROT13 again: Hello World
```

ROT13 is not real encryption — it's used in casual contexts (forums, puzzles, CTFs) to hide content from accidental reading, not to secure it.

### How to translate or delete characters

The `tr` command translates (replaces) or deletes characters from standard input. It reads from a pipe, not directly from a filename.

```bash
[command] | tr 'A-Z' 'a-z'     # convert uppercase to lowercase
[command] | tr -d ' '          # delete spaces
```

`tr` takes two character sets: characters from set 1 are replaced by the matching character in set 2 (position by position).

### How to perform ROT13 with `tr`

To rotate letters by 13 positions, map each letter to the one 13 places later in the alphabet. Since `tr` substitutes character by character, you provide:

- **Set 1:** the original alphabet (`A-Za-z`)
- **Set 2:** the rotated alphabet (`N-ZA-Mn-za-m`)

```bash
cat [file name] | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**How the mapping works:**

|Set 1 (input)|Set 2 (output)|
|---|---|
|`A B C ... M`|`N O P ... Z`|
|`N O P ... Z`|`A B C ... M`|
|`a b c ... m`|`n o p ... z`|
|`n o p ... z`|`a b c ... m`|

Digits and symbols are left alone because they don't appear in set 1.

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
ssh -p 2220 bandit11@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 10 → 11.

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

`file` reports the file as ASCII text. Looking at `cat data.txt`, the content is a readable sentence, but the letters look scrambled — something like:

```
Gur cnffjbeq vf <ENCODED_PASSWORD>
```

This is the goal's hint in action: every letter has been rotated by 13 positions (ROT13).

### 4. Decode the ROT13 text

Pipe the file's content through `tr` to rotate every letter back by 13 positions:

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**How this command works:**

- `cat data.txt` — outputs the encoded text.
- `|` — pipes that output as input to `tr`.
- `tr 'A-Za-z' 'N-ZA-Mn-za-m'` — substitutes each letter with the one 13 positions later in the alphabet.

The output will be a readable sentence containing the password for `bandit12`.

Example:

```
The password is <PASSWORD>
```

---

## Reference

- [Bandit Level 12 Page](https://overthewire.org/wargames/bandit/bandit12.html)