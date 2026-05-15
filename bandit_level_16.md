# Bandit Level 16 → Level 17

## Goal

The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don't. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

**Helpful note:** Getting "DONE", "RENEGOTIATING" or "KEYUPDATE"? Read the "CONNECTED COMMANDS" section in the manpage.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit16`
- **Password:** the password obtained from Level 15 → 16

---

## Hint

- `ssh`
- `telnet`
- `nc`
- `ncat`
- `socat`
- `openssl`
- `s_client`
- `nmap`
- `netstat`
- `ss`

---

## Solution

### How to log in with SSH

```bash
ssh [user]@[host] -p [port]
ssh -i [key file] [user]@[host] -p [port]   # use a private key
```

### How to scan a port range with `nmap`

The `nmap` command can scan a range of ports to find which ones are open. This is the core technique of this level.

```bash
nmap -p [start]-[end] [host]              # scan a port range
nmap -p 31000-32000 localhost             # scan ports 31000 to 32000
```

To also detect what service is running and whether it uses TLS:

```bash
nmap -sV -p [start]-[end] [host]
nmap -sV -p 31000-32000 localhost
```

The `-sV` flag is essential here. It identifies the service type and shows TLS-wrapped services with an `ssl/` prefix:

```
PORT      STATE SERVICE     VERSION
31518/tcp open  ssl/echo
31790/tcp open  ssl/unknown
31960/tcp open  echo
```

### How to connect to a plain TCP service

For ports that do **not** use TLS, use `nc` or `telnet`.

```bash
nc [host] [port]
echo "data" | nc [host] [port]
```

### How to connect to a TLS service with `openssl s_client`

For ports that use TLS, use `openssl s_client`.

```bash
openssl s_client -connect [host]:[port]
openssl s_client -connect [host]:[port] -quiet           # suppress cert output
openssl s_client -connect [host]:[port] -ign_eof         # don't exit on EOF
openssl s_client -connect [host]:[port] -nocommands      # disable control letters
```

**Reminder about CONNECTED COMMANDS:**

While connected, single-letter inputs (`Q`, `R`, `B`, `K`) on their own line are intercepted as control commands instead of being sent. Use `-nocommands` to disable this.

### How to identify echo servers vs the real server

According to the goal, all but one of the open ports are **echo servers** — they simply send back whatever you send. Only one server responds with the next credentials.

**Behavior comparison:**

|Server type|What it returns when you send a password|
|---|---|
|Echo server|The exact same text you sent (echo)|
|Real server|A different response: an error message OR an SSH private key (the credentials)|

In `nmap -sV` output, the real server often appears as **`ssl/unknown`** rather than `ssl/echo` — because its response doesn't match the standard echo pattern that nmap recognizes.

When the password is sent to the real server, it returns an **SSH private key** instead of a plain text password. The credentials for `bandit17` come as a key block.

### How to save and use an SSH private key

When the real server returns a private key, save it to a file, restrict its permissions, and use it to log in as `bandit17`.

```bash
# Create a working directory (home dir is read-only)
mkdir /tmp/[workspace]
cd /tmp/[workspace]

# Save the key (paste it into a file)
nano sshkey.private        # or use vi / vim

# Restrict permissions (required by SSH)
chmod 600 sshkey.private

# Log in as bandit17 with the key (run from your local machine)
ssh -i sshkey.private bandit17@bandit.labs.overthewire.org -p 2220
```

### How to use `ncat`

`ncat` is an enhanced `nc` from the Nmap project. It supports TLS natively.

```bash
ncat [host] [port]            # plain TCP
ncat --ssl [host] [port]      # TLS connection
```

### How to use `socat`

`socat` is a flexible relay tool supporting many protocol combinations.

```bash
socat - OPENSSL:[host]:[port],verify=0
```

### How to check listening ports with `netstat` and `ss`

```bash
netstat -tlnp                 # TCP listening sockets
ss -tlnp                      # modern replacement
ss -tan | grep LISTEN         # filter listening sockets
```

These are useful to see what is listening from within the same machine (without scanning).

---

## Steps

### 1. Log in with SSH as bandit16

```bash
ssh -p 2220 bandit16@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 15 → 16.

### 2. Scan the port range 31000–32000

```bash
nmap -sV -p 31000-32000 localhost
```

Example output:

```
PORT      STATE SERVICE     VERSION
31046/tcp open  echo
31518/tcp open  ssl/echo
31691/tcp open  echo
31790/tcp open  ssl/unknown        ← target candidate
31960/tcp open  echo
```

The `-sV` flag both confirms open ports and identifies TLS vs plain TCP. The scan can take a few minutes because nmap probes each port to identify the service.

### 3. Identify the real server

Most ports show `echo` or `ssl/echo` — these are echo servers that simply reflect input back.

The port labeled **`ssl/unknown`** stands out. Its response does not match the echo pattern, suggesting it is the real server. In the `nmap -sV` output, the "fingerprint" section even shows a hint like:

```
"Wrong! Please enter the correct current password."
```

This is the response the server returns when it receives an incorrect password — clearly not an echo server.

### 4. Connect to the target port with `openssl s_client`

Because the target uses TLS, plain `nc` will not work. Use `openssl s_client`:

```bash
openssl s_client -connect localhost:31790 -nocommands
```

The `-nocommands` option is important: it prevents `openssl s_client` from interpreting single-letter input lines (like `Q`, `R`, `B`, `K`) as control commands. Without it, pasting a password containing such characters can trigger `RENEGOTIATING` or `KEYUPDATE` and break the flow.

After the TLS handshake completes, the connection is interactive.

### 5. Submit the bandit16 password

Paste the bandit16 password and press Enter.

The server responds with an SSH private key, formatted like:

```
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEA...
...
-----END RSA PRIVATE KEY-----
```

This is the credential for `bandit17`.

### 6. Save the key into a file

Create a working directory under `/tmp` and save the key:

```bash
mkdir /tmp/[workspace]
cd /tmp/[workspace]
nano sshkey.private17        # paste the full key block, save and exit
```

Make sure the file includes both the `-----BEGIN-----` and `-----END-----` lines.

### 7. Set proper permissions on the key

```bash
chmod 600 sshkey.private17
```

If permissions are looser, SSH will refuse to use the key and fall back to password authentication.

### 8. Log in as bandit17 from the local machine

OverTheWire blocks SSH between levels via `localhost`, so the key must be used **from your own local machine**. Either:

- Transfer the key with `scp`, then log in, OR
- Re-create the same key file on your local machine and log in

```bash
ssh -i sshkey.private17 -p 2220 bandit17@bandit.labs.overthewire.org
```

If permissions are correct and the key matches, the login succeeds without a password prompt.

### 9. Confirm access

After logging in, `whoami` should return `bandit17`. The password for bandit17 is now in `/etc/bandit_pass/bandit17` (readable as bandit17).

```bash
cat /etc/bandit_pass/bandit17
```

---

## Notes

- This level combines almost everything learned in Bandit 13–16: SSH key authentication, plain TCP vs TLS services, port scanning with `nmap`, and the `openssl s_client` control-command pitfall.
- The "real" server is identifiable because it is the only one that does **not** echo input back. Either inspect `nmap -sV` output for unusual fingerprints, or manually probe each candidate port and look for non-echo behavior.

---

## Reference

- [Bandit Level 17 Page](https://overthewire.org/wargames/bandit/bandit17.html)
- [Port scanner on Wikipedia](https://en.wikipedia.org/wiki/Port_scanner)