# Bandit Level 9 → Level 10

## Goal

The password for the next level is stored in the file `data.txt` in one of the few human-readable strings, preceded by several `=` characters.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit9`
- **Password:** the password obtained from Level 8

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
ssh -p 2220 bandit9@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 8 → 9.

### 2. List the files in the current directory

```bash
ls
```

After running `ls`, you will see a single file named `data.txt`.

### 3. Check data.txt's information with the `file` command

```bash
file data.txt
```

The output is:

```
data.txt: data
```

This means the `file` command could not identify the file's format — it's treated as generic binary data. The file contains a mix of binary noise and a few human-readable strings, so we cannot simply read it with `cat`.

### 4. Extract the password using `strings` and `grep`

We use `strings` to pull out the readable text from the binary file, then filter for the lines preceded by `=` characters.

```bash
strings data.txt | grep "===="
```

**How this command works:**

- `strings data.txt` — extracts sequences of printable characters from the binary file.
- `grep "===="` — keeps only the lines that contain four or more `=` characters (the marker mentioned in the goal).

The output reveals the password for `bandit10`.

---

## Reference

- [Bandit Level 10 Page](https://overthewire.org/wargames/bandit/bandit10.html)