# Bandit Level 0 → Level 1

## Goal

The goal of this level is to log into the game using SSH.

---
## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit0`
- **Password:** `bandit0`

---
## Hint

- `ssh`

---
## Solution

### How to use SSH


```bash
ssh -p [port] [username]@[host]
```

---
## Steps

### 1. login with ssh

From the problem, we can identify four pieces of information:

1. Host: `bandit.labs.overthewire.org`
2. Port: `2220`
3. Username: `bandit0`
4. Password: `bandit0`

So we can connect via SSH with the following command:

```bash
ssh -p 2220 bandit0@bandit.labs.overthewire.org
```

When prompted, enter the password `bandit0`.

### 2. After Successful Login

Once you have logged in successfully, type `ls` to list the files in the current directory.

```bash
ls
```

You will see a file named `readme`. Use the `cat` command to read its contents, which will reveal the password for the next level.

```bash
cat readme
```

The output will display the password you need to log in as `bandit1` in the next level.

---
## Reference

- [Bandit Level 1 Page](https://overthewire.org/wargames/bandit/bandit1.html)
