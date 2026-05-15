# Bandit Level 17 → Level 18

## Goal

There are 2 files in the home directory: `passwords.old` and `passwords.new`. The password for the next level is in `passwords.new` and is the only line that has been changed between `passwords.old` and `passwords.new`.

**NOTE:** if you have solved this level and see 'Byebye!' when trying to log into bandit18, this is related to the next level, bandit19.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit17`
- **Password:** the password obtained from Level 16 → 17 (via SSH key authentication)

---

## Hint

- `cat`
- `grep`
- `ls`
- `diff`

---

## Solution

### How to list files

```bash
ls                  # list files in current directory
ls -la              # also show hidden files and details
```

### How to view a file's contents

```bash
cat [file name]
```

### How to compare two files with `diff`

The `diff` command compares two files line by line and reports the differences.

```bash
diff [file1] [file2]
```

**Output format:**

```
42c42
< line from file1
---
> line from file2
```

- `42c42` — line 42 of file1 was **c**hanged into line 42 of file2
- `<` — content from the **first file** (left argument)
- `>` — content from the **second file** (right argument)

**Other change indicators:**

|Symbol|Meaning|
|---|---|
|`c`|changed (e.g., `5c5`)|
|`a`|added (e.g., `5a6`)|
|`d`|deleted (e.g., `5d4`)|

### How `<` and `>` map to files

The mapping depends on **the order of arguments** in the `diff` command:

```bash
diff passwords.old passwords.new
# <  →  passwords.old
# >  →  passwords.new

diff passwords.new passwords.old
# <  →  passwords.new
# >  →  passwords.old
```

The password for `bandit18` is the line that exists in `passwords.new`. Whichever side of the diff corresponds to `passwords.new` is the answer.

---

## Steps

### 1. Log in as bandit17

Since the credentials for bandit17 are an SSH private key (received from Level 16 → 17), log in with key authentication from your local machine:

```bash
ssh -i sshkey.private17 bandit17@bandit.labs.overthewire.org -p 2220
```

### 2. List the files in the home directory

```bash
ls
```

You should see two files:

```
passwords.new
passwords.old
```

### 3. Compare the two files with `diff`

```bash
diff passwords.old passwords.new
```

Example output:

```
42c42
< <OLD_PASSWORD_LINE>
---
> <NEW_PASSWORD_LINE>
```

- The line marked with `>` belongs to `passwords.new` — this is the password for `bandit18`.
- The line marked with `<` is the old password from `passwords.old` and is **not** the answer.

If the arguments are reversed (`diff passwords.new passwords.old`), the meaning of `<` and `>` flips accordingly.

### 4. Extract only the password (optional)

To programmatically extract just the changed line from `passwords.new`:

```bash
diff passwords.old passwords.new | grep "^>" | cut -d' ' -f2
```

- `grep "^>"` keeps only lines starting with `>`
- `cut -d' ' -f2` splits by space and takes the second field (the password itself)

### 5. Note about logging in as bandit18

After verifying the password, attempting to SSH into bandit18 may immediately disconnect with the message `Byebye!`. This is **expected** — the login itself succeeded, but bandit18's shell is configured to exit on connection. Bypassing this is the focus of Level 18 → 19, not this level.

---

## Notes

- `diff` is the standard tool for comparing files in Linux/Unix. It is widely used in version control, code review, and forensics.
- `<` always represents the **left-hand file** in the command, regardless of which file is "older" or "newer." Always check the argument order before interpreting the output.
- For larger comparisons, `diff -u` produces unified diff output (the same format used by Git), which is easier to read.

---

## Reference

- [Bandit Level 18 Page](https://overthewire.org/wargames/bandit/bandit18.html)
- `man diff` for the full option list