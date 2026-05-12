# Bandit Level 2 → Level 3

## Goal

The goal of this level is to get the password for `bandit3`.

---
## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit2`
- **Password:** the password obtained from Level 1

---
## Hint

- `ssh`
- `ls`
- `cd`
- `cat`
- `file`
- `du`
- `find`

---
## Solution

### How to use SSH

```bash
ssh -p [port] [username]@[host]
```

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

```bash
file [file name]
du [file name]
```

### How to find files

```bash
find
```

### How to read a file whose name contains spaces and starts with `-`

This filename has two problems at the same time:

1. **Spaces in the name** — the shell will split the filename into multiple arguments unless you quote it or escape each space with a backslash (`\`).
2. **Starts with a dash (`-`)** — commands like `cat` will interpret the dash as a command-line option instead of a filename.

To handle both, wrap the filename in quotes (to deal with spaces) **and** prefix it with `./` (to deal with the leading dash):

```bash
cat "./--spaces in this filename--"
```

Alternatively, you can use input redirection, which avoids passing the filename as an argument to `cat` at all:

```bash
cat < "--spaces in this filename--"
```

You can also use Tab completion: type the first few characters and press `Tab`, and the shell will automatically escape the spaces for you.

---
## Steps

### 1. Log in with SSH

From the problem, we can identify four pieces of information:

1. Host: `bandit.labs.overthewire.org`
2. Port: `2220`
3. Username: `bandit2`
4. Password: the password obtained from Level 1

So we can connect via SSH with the following command:

```bash
ssh -p 2220 bandit2@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 1.

### 2. List the files in the current directory

```bash
ls
```

After running `ls`, you will see a single file named `--spaces in this filename--`.

### 3. Read the file named `--spaces in this filename--`

The filename has two issues: it contains spaces, and it starts with a dash (`-`). Running `cat --spaces in this filename--` directly will fail because `cat` interprets `--spaces` as an option and the rest as separate arguments.

Use one of the following commands to read the file:

```bash
cat "./--spaces in this filename--"
```

The `./` prefix tells the shell that the filename refers to a file in the current directory, not an option. The double quotes keep the spaces from splitting the filename into multiple arguments.

Alternatively, you can use input redirection:

```bash
cat < "--spaces in this filename--"
```

The output will display the password needed to log in as `bandit3` in the next level.

---
## Reference

- [Bandit Level 3 Page](https://overthewire.org/wargames/bandit/bandit3.html)