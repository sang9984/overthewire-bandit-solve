# Bandit Level 13 → Level 14

## Goal

The password for the next level is stored in `/etc/bandit_pass/bandit14` and can only be read by user `bandit14`. For this level, you don't get the next password, but you get a private SSH key that can be used to log into the next level. **Note:** `localhost` is a hostname that refers to the machine you are working on.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit13`
- **Password:** the password obtained from Level 12 → 13

---

## Hint

- `ssh`
- `scp`
- `umask`
- `chmod`
- `cat`
- `nc`
- `install`

---

## Solution

### How to log in with SSH using a private key

The `ssh` command opens a secure shell connection to a remote machine. Normally it prompts for a password, but you can authenticate with a **private key** instead by using the `-i` option.

```bash
ssh -i [key file] [user]@[host] -p [port]
```

- `-i [key file]` — use this private key for authentication.
- `-p [port]` — connect to a specific port (Bandit uses `2220`).

### How to copy files over SSH

The `scp` (Secure CoPy) command copies files between machines using the SSH protocol.

```bash
scp -P [port] [source] [user]@[host]:[destination]   # local → remote
scp -P [port] [user]@[host]:[source] [destination]   # remote → local
```

**Note:** `scp` uses an **uppercase `-P`** for the port, while `ssh` uses a lowercase `-p`.

### How to view a file's contents

The `cat` command prints the contents of a file to standard output.

```bash
cat [file name]
```

### How to set the default permissions of new files

The `umask` command defines a mask that is **subtracted** from the default permissions whenever a new file or directory is created. It controls which permissions are removed by default.

```bash
umask              # show the current umask value
umask [value]      # set a new umask (e.g., umask 077)
```

How the mask works:

- File default permissions are `666` (rw-rw-rw-).
- Directory default permissions are `777` (rwxrwxrwx).
- The umask value is subtracted from those defaults.

|umask|Resulting file permissions|Meaning|
|---|---|---|
|`022`|`644` (rw-r--r--)|Owner read/write, others read|
|`077`|`600` (rw-------)|Owner read/write only|

### How to change a file's permissions

The `chmod` (CHange MODe) command changes the access permissions of a file or directory. SSH private keys must have strict permissions (`600`) or `ssh` will refuse to use them.

**Numeric (octal) form:**

```bash
chmod [mode] [file name]
chmod 600 [key file]       # owner read/write only
chmod 755 [script]         # owner all, others read/execute
```

**Permission digit values:**

- `4` = read (`r`)
- `2` = write (`w`)
- `1` = execute (`x`)

The three digits represent **owner / group / others**, in that order.

**Symbolic form:**

```bash
chmod u+x [file]           # add execute for user (owner)
chmod g-w [file]           # remove write for group
chmod o-r [file]           # remove read for others
```

### How to use netcat (`nc`)

The `nc` (NetCat) command reads and writes data over TCP or UDP connections. It's often called the "Swiss Army knife" of networking. It is used in later Bandit levels to talk to network services.

```bash
nc [host] [port]           # connect to a TCP service
nc -l [port]               # listen on a port (server mode)
nc -zv [host] [port]       # check if a port is open
```

### How to install (copy) a file with specific permissions

The `install` command copies files like `cp`, but with the added ability to set permissions, ownership, and create destination directories in a single step.

```bash
install -m [mode] [source] [destination]   # copy with specific permissions
install -d [directory]                     # create a directory
install -m 600 [key file] /tmp/mykey       # copy and set permissions to 600
```

This is useful when you need both `cp` and `chmod` at once.

---

## Steps

> **Important:** OverTheWire blocks SSH connections from one level to another via `localhost`. The private key must be transferred to your **own local machine** and used from there to log into `bandit14`.

### 1. Log in with SSH as bandit13

```bash
ssh -p 2220 bandit13@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 12 → 13.

### 2. Locate the private key

```bash
ls
```

You will see a file named `sshkey.private` in the home directory. This is the key needed to log in as `bandit14`.

### 3. Create a working directory under `/tmp` and copy the key there

The home directory is read-only, and the original key file is not owned by `bandit13`, so its permissions cannot be changed in place. Create a temporary working directory and copy the key into it.

```bash
mkdir /tmp/[your_workspace]
cp sshkey.private /tmp/[your_workspace]/
cd /tmp/[your_workspace]
```

After copying, the file is owned by `bandit13`, so permissions can now be modified.

### 4. (Optional) Verify and fix the key's permissions

```bash
ls -l sshkey.private
chmod 600 sshkey.private
```

SSH requires private keys to be readable only by the owner (`600`). If looser, SSH will refuse to use the key.

### 5. Log out of the bandit server

```bash
exit
```

You need to be back on your **local machine** for the next step, because OverTheWire blocks `localhost`-to-`localhost` SSH between levels.

### 6. Copy the key from the bandit server to your local machine using `scp`

Run this command on your **local terminal**, not inside the bandit server:

```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:/tmp/[your_workspace]/sshkey.private [your_local_path]
```

When prompted, enter the password for `bandit13`.

### 7. Log in as bandit14 from your local machine using the key

Still on your local terminal, use the downloaded key to log in directly as `bandit14`:

```bash
ssh -i [your_local_path]/sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```

**How this command works:**

- `-i [key file]` — authenticate using this private key.
- `bandit14@bandit.labs.overthewire.org` — log in as `bandit14` on the bandit server.
- `-p 2220` — connect to port 2220.

No password is required because the SSH key handles authentication.

### 8. Read the password file

Once logged in as `bandit14`, read the password file:

```bash
cat /etc/bandit_pass/bandit14
```

The output reveals the password for `bandit14`.

---

## Reference

- [Bandit Level 14 Page](https://overthewire.org/wargames/bandit/bandit14.html)