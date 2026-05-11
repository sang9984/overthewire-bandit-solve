# Bandit Level 1 → Level 2

## Goal

The goal of this level is to get the password for `bandit2`.

---
## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit1`
- **Password:** the password obtained from Level 0

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

### How to read a file whose name starts with `-`

A filename starting with a dash (`-`) is interpreted as a command-line option, so `cat -` will not work. You need to tell the shell that the dash is a filename, not an option. There are two common ways:

```bash
cat ./-
```

```bash
cat < -
```

---
## Steps

### 1. Log in with SSH

From the problem, we can identify four pieces of information:

1. Host: `bandit.labs.overthewire.org`
2. Port: `2220`
3. Username: `bandit1`
4. Password: the password obtained from Level 0

So we can connect via SSH with the following command:

```bash
ssh -p 2220 bandit1@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 0.

### 2. List the files in the current directory

```bash
ls
```

After running `ls`, you will see a single file named `-`.

### 3. Read the file named `-`

Since the filename `-` is treated as an option by most commands, `cat -` will not work (it reads from standard input instead). Use one of the following commands to read the file:

```bash
cat ./-
```

The `./` prefix explicitly tells the shell that `-` is a filename in the current directory.

Alternatively, you can use input redirection:

```bash
cat < -
```

The output will display the password needed to log in as `bandit2` in the next level.

---
## Reference

- [Bandit Level 2 Page](https://overthewire.org/wargames/bandit/bandit2.html)