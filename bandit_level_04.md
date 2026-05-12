# Bandit Level 4 → Level 5

## Goal

The goal of this level is to get the password for `bandit5`.

---
## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit4`
- **Password:** the password obtained from Level 3

---
## Hint

- `ls`
- `cd`
- `cat`
- `file`
- `du`
- `find`
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

### How to check a file's information

The `file` command identifies the type of a file (text, binary, image, etc.):

```bash
file [file name]
```

The `du` command shows disk usage:

```bash
du [file name]
```

### How to find files recursively

The `find` command lists all files recursively, starting from the given directory. The `-type f` option limits the search to regular files (not directories):

```bash
find . -type f
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

### How to run a command on every file found by `find`

The `-exec` option runs a command on each file that `find` matches. The `{}` placeholder is replaced by the current filename, and `\;` marks the end of the command:

```bash
find . -type f -exec [command] {} \;
```

---
## Steps

### 1. Log in with SSH

```bash
ssh -p 2220 bandit4@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 3.

### 2. List the files in the current directory

```bash
ls
```

After running `ls`, you will see a single directory named `inhere`.

### 3. Move into the `inhere` directory

```bash
cd inhere
```

### 4. List the files inside `inhere`

```bash
ls
```

You will see multiple files (for example, `-file00` through `-file09`). Only one of them is the password file, and the rest contain non-readable data. We need a way to identify which one is human-readable text.

### 5. Find the password file by file type

Since the password is plain text, we can use the `file` command to detect which file is **ASCII text**. To check every file at once, combine `find`, `-exec file`, and `grep`:

```bash
find . -type f -exec file {} \; | grep "ASCII text"
```

**How this command works:**

- `find . -type f` — find every regular file in the current directory and below.
- `-exec file {} \;` — run the `file` command on each one. `{}` is replaced by the filename, and `\;` ends the `-exec` command.
- `| grep "ASCII text"` — pipe the output to `grep`, which keeps only the lines that contain `ASCII text`.

The output will look something like this:

```
./-file07: ASCII text
```

### 6. Read the identified file

The filename starts with a dash (`-`), so prefix it with `./` to prevent `cat` from interpreting it as an option:

```bash
cat ./-file07
```

The output will display the password needed to log in as `bandit5` in the next level.

---
## Reference

- [Bandit Level 5 Page](https://overthewire.org/wargames/bandit/bandit5.html)