# Bandit Level 19 → Level 20

## Goal

To gain access to the next level, you should use the setuid binary in the home directory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (`/etc/bandit_pass`), after you have used the setuid binary.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit19`
- **Password:** the password obtained from Level 18 → 19

---

## Hint

- (Not specified by the challenge — but `ls`, `cat`, `./` execution, and understanding of setuid are key)

---

## Solution

### What is a setuid binary

**setuid** (Set User ID on execution) is a special permission bit on executable files. When a binary has the setuid bit set, anyone who executes it runs the program with the **owner's privileges**, not their own.

**How to identify a setuid file:**

```bash
ls -l [file]
```

Example output:

```
-rwsr-x--- 1 bandit20 bandit19 ... bandit20-do
   ↑
   's' instead of 'x' in the owner's execute position
```

- The `s` in the owner's execute slot indicates the setuid bit.
- Owner = `bandit20`, so executing this file runs with bandit20's permissions, even when launched by bandit19.

### Why setuid exists

Some operations require elevated permissions but should be available to regular users in a controlled way. For example, `passwd` is setuid-root because changing one's own password requires writing to a system file. The setuid mechanism gives users limited, controlled access to privileged actions.

### Why setuid matters in security

setuid binaries are a major **privilege escalation surface**. In penetration testing, one of the first things to check on a target system is the list of setuid binaries:

```bash
find / -perm -4000 -type f 2>/dev/null
```

A misconfigured or vulnerable setuid binary can let a low-privilege user execute commands as a higher-privilege user (often root).

### How to execute a file in the current directory

Files in the current directory aren't on the system `PATH`, so they must be invoked with `./` prefix:

```bash
./[binary name]
```

For example:

```bash
./bandit20-do
```

### How `bandit20-do` works

Running `bandit20-do` without arguments prints its usage:

```
Run a command as another user.
  Example: ./bandit20-do id
```

The binary takes a command as its argument and executes it as **bandit20**, because the setuid bit is set on the file (and the file is owned by bandit20).

**Verification:**

```bash
./bandit20-do whoami
```

Output:

```
bandit20
```

This confirms that, although you are logged in as `bandit19`, commands launched through this binary run with bandit20's identity.

### How to read the bandit20 password

The password file for bandit20 is at `/etc/bandit_pass/bandit20`, but it is readable only by user bandit20. Use `bandit20-do` to run `cat` on that file with bandit20's privileges:

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

---

## Steps

### 1. Log in as bandit19

```bash
ssh -p 2220 bandit19@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 18 → 19.

### 2. Identify the setuid binary in the home directory

```bash
ls -l
```

Example output:

```
-rwsr-x--- 1 bandit20 bandit19 ... bandit20-do
```

The `s` bit and the owner `bandit20` confirm this is the setuid binary the challenge refers to.

### 3. Run the binary without arguments to see its usage

```bash
./bandit20-do
```

The binary prints something like:

```
Run a command as another user.
  Example: ./bandit20-do id
```

### 4. Verify it runs as bandit20

```bash
./bandit20-do whoami
```

Output:

```
bandit20
```

This confirms that any command run through this wrapper executes with bandit20's permissions.

### 5. Read the bandit20 password file

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

The contents of `/etc/bandit_pass/bandit20` are printed to the terminal. This is the password for `bandit20`.

---

## Notes

- This level introduces the **setuid mechanism**, a critical concept in Unix-like privilege management and in security.
- The pattern `./binary command` (passing a command for the setuid binary to run) is reminiscent of `sudo command` — both are wrappers that execute commands with elevated privileges. The difference is that `sudo` uses `/etc/sudoers` to control who can do what, while a setuid binary just runs as its owner for anyone who can execute it.
- In real-world security work, a misconfigured setuid binary (e.g., one that lets users specify an arbitrary command, like in this challenge) is a textbook privilege escalation vulnerability. Real-world setuid binaries should restrict what commands they accept to a tightly controlled list.
- Useful follow-up reading: GTFOBins (https://gtfobins.github.io/) catalogs Unix binaries that can be abused for privilege escalation when setuid is misapplied.

---

## Reference

- [Bandit Level 20 Page](https://overthewire.org/wargames/bandit/bandit20.html)
- `man chmod` — see the section on setuid
- [Setuid on Wikipedia](https://en.wikipedia.org/wiki/Setuid)
- [GTFOBins](https://gtfobins.github.io/) — collection of Unix binaries that can be abused for privilege escalation