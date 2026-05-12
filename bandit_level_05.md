# Bandit Level 5 → Level 6

## Goal

The goal of this level is to get the password for `bandit6`.

The password file is stored somewhere under the `inhere` directory and meets all of the following conditions:

- human-readable (plain text)
- 1033 bytes in size
- not executable

---
## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit5`
- **Password:** the password obtained from Level 4

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

The `find` command lists all files recursively. The `-type f` option limits the search to regular files (not directories):

```bash
find . -type f
```

### How to filter files by size with `find`

The `-size` option filters by file size. The suffix specifies the unit:

|Suffix|Meaning|
|---|---|
|`c`|bytes|
|`k`|kilobytes|
|`M`|megabytes|

```bash
find . -size 1033c       # exactly 1033 bytes
find . -size +1k         # larger than 1 KB
find . -size -100c       # smaller than 100 bytes
```

### How to filter files by permission with `find`

The `-executable` option matches executable files. The `!` operator negates a condition, so `! -executable` matches files that are **not** executable:

```bash
find . -executable        # executable files
find . ! -executable      # non-executable files
find . -readable          # readable files
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
ssh -p 2220 bandit5@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 4.

### 2. List the files in the current directory

```bash
ls
```

You will see a directory named `inhere`.

### 3. Move into the `inhere` directory

```bash
cd inhere
```

### 4. Inspect the structure

```bash
ls
```

You will see many subdirectories named `maybehere00` through `maybehere19`. The password file is hidden somewhere inside one of these subdirectories, so we need to search recursively.

### 5. Find the password file by combining all three conditions

The problem gives us three conditions to identify the password file: it must be human-readable text, exactly 1033 bytes in size, and not executable. We can combine all of these in a single `find` command:

```bash
find . -type f -size 1033c ! -executable -exec file {} \; | grep "ASCII text"
```

**How this command works:**

- `find .` — start the search from the current directory.
- `-type f` — match only regular files (skip directories).
- `-size 1033c` — match only files exactly 1033 bytes in size (`c` means bytes).
- `! -executable` — exclude executable files (`!` is the NOT operator).
- `-exec file {} \;` — run the `file` command on each matching file. `{}` is replaced by the filename, and `\;` ends the `-exec` command.
- `| grep "ASCII text"` — pipe the output to `grep`, which keeps only the lines that contain `ASCII text`, confirming the file is plain text.

The output will look something like this:

```
./maybehere07/.file2: ASCII text, with very long lines
```

### 6. Read the identified file

Use `cat` with the path returned by the previous step:

```bash
cat ./maybehere07/.file2
```

The output will display the password needed to log in as `bandit6` in the next level.

---
## Deep Dive: Why is `|` used only before `grep`?

A common question about this command is: why do we use a pipe (`|`) before `grep`, but not before `! -executable`? The answer lies in understanding the difference between **a command's options** and **a pipe between commands**.

### `find`'s options vs. a pipe

|Concept|`find`'s options (`-type`, `-size`, `!`, `-exec`)|Pipe (`\|`)|
|---|---|---|
|Where it works|**Inside** the `find` command|**Between** two separate commands|
|What it does|Combines filter conditions and actions|Sends one command's output to another's input|
|What follows it|Arguments that `find` understands|Another independent executable command|

`-type f`, `-size 1033c`, `! -executable`, and `-exec file {} \;` are all **arguments passed to `find`**. They are not separate commands. `find` reads them, combines the filter conditions with AND logic, and then performs the action.

A pipe, on the other hand, takes the **output** of the entire previous command and feeds it as **input** into a completely separate command (like `grep`, `awk`, or `sort`).

### Why `grep` needs a pipe

`grep` is an independent program. It reads text from standard input and prints lines that match a pattern. For `grep` to receive the output of `find ... -exec file {} \;`, we need a pipe to connect them:

```
[ find ... -exec file {} \; ]  →  pipe  →  [ grep "ASCII text" ]
       outputs lines of text                 reads those lines
       like "./maybehere07/.file2:           and filters them
       ASCII text"
```

### Why `! -executable` doesn't need a pipe

`-executable` is not a command — it's a **filter option** that belongs to `find`. It only has meaning inside `find`. Writing this:

```bash
find . -type f -size 1033c | ! -executable -exec file {} \;
```

would not work, because after the pipe, the shell expects a real command, but `! -executable` is just an option name — there is no executable program called `! -executable`.

### A useful analogy

Think of it this way:

- **`find`'s options** are like ingredients in a single recipe. You list them all together inside one cooking step.
- **A pipe** is like passing the finished dish from one chef to another.

```bash
find . -type f -size 1033c ! -executable -exec file {} \;
└────────────────────────┬───────────────────────────┘
              one chef (find) doing everything
              with its ingredients (options)
                              │
                              │  hands the finished dish over
                              ▼
                          | grep "ASCII text"
                          └─────────────────┘
                          next chef (grep)
                          processes the dish
```

### Why can't `grep "ASCII text"` also be a `find` option?

You might wonder why `find` can't filter by "ASCII text" directly. The reason is that `find` only knows about **file metadata** (name, size, type, permissions, modification time). To check whether a file's **content** is ASCII text, you have to actually look inside the file, which is the job of the `file` command.

So the workflow naturally splits into three stages, with each tool doing what it's best at:

1. **`find`** — narrow down candidates by metadata (size, type, permissions).
2. **`file`** — examine each candidate's content to determine its type.
3. **`grep`** — filter the `file` command's output to keep only "ASCII text" matches.

This is the Unix philosophy in action: each small tool does one thing well, and pipes let you combine them into more powerful workflows.

---
## Reference

- [Bandit Level 6 Page](https://overthewire.org/wargames/bandit/bandit6.html)