# Bandit Level 3 → Level 4

## Goal

The goal of this level is to get the password for `bandit4`.

---
## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit3`
- **Password:** the password obtained from Level 2

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

### How to list hidden files

In Linux, files whose names start with a dot (`.`) are hidden by default. The plain `ls` command does not show them. Use the `-a` (all) option to include hidden files:

```bash
ls -a
```

You can also combine it with `-l` for a detailed listing:

```bash
ls -la
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

### How to find hidden files with `find`

The `find` command lists all files recursively, including hidden ones, starting from the current directory:

```bash
find
```

---
## Steps

### 1. Log in with SSH

From the problem, we can identify four pieces of information:

1. Host: `bandit.labs.overthewire.org`
2. Port: `2220`
3. Username: `bandit3`
4. Password: the password obtained from Level 2

So we can connect via SSH with the following command:

```bash
ssh -p 2220 bandit3@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 2.

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

This time, `ls` shows no files. That is because the file inside this directory is **hidden** — its name starts with a dot (`.`), and `ls` does not show such files by default.

### 5. Reveal the hidden file

There are two common ways to find hidden files.

**Option A — `ls` with the `-a` option:**

```bash
ls -a
```

The output will look like this:

```
.  ..  ...Hiding-From-You
```

- `.` represents the current directory.
- `..` represents the parent directory.
- `...Hiding-From-You` is the hidden file we are looking for.

**Option B — `find` command:**

```bash
find
```

The output will look like this:

```
.
./...Hiding-From-You
```

`find` lists every file recursively, including hidden ones.

---
### 6. Read the hidden file

The filename starts with three dots, but it does not contain spaces, so you can read it directly with `cat`:

```bash
cat ...Hiding-From-You
```

If you want to be safe (since the name looks like it could be confused with options or relative paths), you can also use:

```bash
cat ./...Hiding-From-You
```

The output will display the password needed to log in as `bandit4` in the next level.

---
## Reference

- [Bandit Level 4 Page](https://overthewire.org/wargames/bandit/bandit4.html)