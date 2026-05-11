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
### Steps

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

---
## Reference

- [Bandit Level 1 Page](https://overthewire.org/wargames/bandit/bandit1.html)
