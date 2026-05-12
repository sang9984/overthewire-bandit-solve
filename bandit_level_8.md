# Bandit Level 8 → Level 9

## Goal

The password for the next level is stored in the file `data.txt` and is the only line of text that occurs only once.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit8`
- **Password:** the password obtained from Level 7

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

The `-u` option of `uniq` prints only lines that are **not repeated**. When combined with `sort`, this gives you items that appear exactly once in the file.

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
ssh -p 2220 bandit8@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 7 → 8.

### 2. List the files in the current directory

```bash
ls
```

After running `ls`, you will see a single ASCII text file `data.txt`.

### 3. Check data.txt's size with the `du` command

```bash
du data.txt
```

The output shows that this file contains a large amount of data. Since we need to find the only line that occurs once, we'll use `sort` and `uniq` together.

### 4. Get the only line of text that occurs once

```bash
sort data.txt | uniq -u
```

**How this command works:**

- `sort data.txt` — sorts the lines so that identical lines become adjacent (required for `uniq` to detect duplicates).
- `uniq -u` — outputs only the lines that are not repeated.

The output will be a single line: the password for `bandit9`.

---

## Reference

- [Bandit Level 9 Page](https://overthewire.org/wargames/bandit/bandit9.html)