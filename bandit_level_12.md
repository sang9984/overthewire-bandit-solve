# Bandit Level 12 → Level 13

## Goal

The password for the next level is stored in the file `data.txt`, which is a hexdump of a file that has been repeatedly compressed. For this level, it may be useful to create a working directory under `/tmp`. In that directory, copy the data file using `cp`, and rename it using `mv` (read the manpages!).

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit12`
- **Password:** the password obtained from Level 11 → 12

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
- `mkdir`
- `cp`
- `mv`
- `file`

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

### How to identify a hex dump

When `file` reports a file as `ASCII text`, the content may actually be a **hex dump** of binary data. You need to inspect the content with `head` or `cat` to tell which format it is.

There are two common hex formats. Recognizing which one you have determines which decoding command to use.

#### xxd hex dump format (with offsets)

**Appearance:**

```
00000000: 1f8b 0808 00da cf69 0203 6461 7461 322e  .......i..data2.
00000010: 6269 6e00 0136 02c9 fd42 5a68 3931 4159  bin..6...BZh91AY
```

**Characteristics:**

- Each line starts with an **8-digit hex offset** followed by `:`
- Middle section: hex bytes grouped in pairs (`1f8b 0808`)
- Right section: ASCII representation (non-printable bytes shown as `.`)

**Decode command:**

```bash
xxd -r [file name] > [output file]
```

#### Plain hex format (hex only)

**Appearance:**

```
1f8b08000000000000034b4c4a06000aaef940cb6e08
```

**Characteristics:**

- Only **`0-9` and `a-f`** (or `A-F`) characters
- No offsets, no ASCII column

**Decode command:**

```bash
xxd -r -p [file name] > [output file]
```

#### Quick comparison table

|Format|First few characters|Decode command|
|---|---|---|
|xxd hex dump|`00000000: 1f8b ...`|`xxd -r`|
|Plain hex|`1f8b0800...`|`xxd -r -p`|

### How to create a directory

The `mkdir` command creates a new directory. In this level, the home directory is not writable, so we use `/tmp` as a working space.

```bash
mkdir [directory name]
mkdir /tmp/[your_workspace]
```

### How to copy a file

The `cp` command copies a file from one location to another.

```bash
cp [source] [destination]
```

### How to rename or move a file

The `mv` command moves a file, or renames it when the source and destination are in the same directory.

```bash
mv [source] [destination]        # move
mv [old name] [new name]         # rename in place
```

In this level, `mv` is used to add the correct file extension (`.gz`, `.bz2`, `.tar`) so the decompression tools recognize the format.

### How to check a file's type

The `file` command identifies a file's format by inspecting its content (magic bytes), not its extension. This is the key tool for figuring out what compression to undo at each step.

```bash
file [file name]
```

Common outputs you will see in this level:

|`file` output|What it means|Next command|
|---|---|---|
|`gzip compressed data`|gzip-compressed|rename to `.gz`, then `gunzip`|
|`bzip2 compressed data`|bzip2-compressed|rename to `.bz2`, then `bunzip2`|
|`POSIX tar archive`|tar archive (bundle)|rename to `.tar`, then `tar -xf`|
|`ASCII text`|plain text (likely the final password)|read with `cat`|

---

## Steps

### 1. Log in with SSH

```bash
ssh -p 2220 bandit12@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 11 → 12.

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

`file` reports the file as `ASCII text`. Looking at `cat data.txt`, the content looks like this:

```
00000000: 1f8b 0808 00da cf69 0203 6461 7461 322e  .......i..data2.
00000010: 6269 6e00 0136 02c9 fd42 5a68 3931 4159  bin..6...BZh91AY
00000020: 2653 5978 ae89 f600 001c 7fff db7d bfef  &SYx.........}..
...
```

This is an **xxd hex dump** (lines start with an 8-digit offset followed by `:`). The actual content is a compressed binary file represented as text.

### 4. Create a working directory in `/tmp`

The home directory is read-only, so we move to `/tmp` to work on the file.

```bash
mkdir /tmp/mywork
cp data.txt /tmp/mywork/
cd /tmp/mywork
```

### 5. Reverse the hex dump

Use `xxd -r` to convert the hex dump back into the original binary file.

```bash
xxd -r data.txt > data
file data
```

The output of `file data` will tell us what kind of compressed file it actually is (for example, `gzip compressed data`).

### 6. Repeatedly decompress until plain text appears

The file has been compressed multiple times using different algorithms. Repeat the following loop until `file` reports `ASCII text`:

1. Check the file type with `file`.
2. Rename the file with the appropriate extension using `mv`.
3. Decompress with the matching tool.

**Examples for each format:**

```bash
# gzip
mv data data.gz
gunzip data.gz
file data

# bzip2
mv data data.bz2
bunzip2 data.bz2
file data

# tar archive
mv data data.tar
tar -xf data.tar
ls                      # check the new file name produced by tar
file <new_file_name>
```

Repeat this loop. The compression layers are typically a mix of `gzip`, `bzip2`, and `tar`, applied 5–7 times in total.

### 7. Read the final file

When `file` finally reports `ASCII text`, read the file with `cat`:

```bash
cat <final_file>
```

The output reveals the password for `bandit13`.

---

## Reference

- [Bandit Level 13 Page](https://overthewire.org/wargames/bandit/bandit13.html)