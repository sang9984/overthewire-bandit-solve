# Bandit Level 14 → Level 15

## Goal

The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit14`
- **Password:** the password obtained from Level 13 → 14

---

## Hint

- `ssh`
- `telnet`
- `nc`
- `openssl`
- `s_client`
- `nmap`

---

## Solution

### How to log in with SSH

The `ssh` command opens a secure shell connection to a remote machine.

```bash
ssh [user]@[host] -p [port]
ssh -i [key file] [user]@[host] -p [port]   # use a private key
```

- `-p [port]` — connect to a specific port (Bandit uses `2220`).
- `-i [key file]` — authenticate with a private key.

### How to connect to a TCP service with telnet

The `telnet` command opens an interactive plain-text TCP connection to a host and port. It is often used to test if a port is open and to send simple text-based commands.

```bash
telnet [host] [port]
```

After connecting, anything you type is sent to the service, and replies are printed back. Use `Ctrl+]` then `quit` to exit.

### How to use netcat (`nc`)

The `nc` (NetCat) command reads and writes data over TCP or UDP connections. It is more flexible than `telnet` and works well for sending data programmatically (e.g., piping a file or `echo` output into a service).

```bash
nc [host] [port]              # connect to a TCP service
nc -l [port]                  # listen on a port (server mode)
nc -zv [host] [port]          # check if a port is open
nc -u [host] [port]           # UDP instead of TCP
```

Sending data through `nc`:

```bash
echo "data" | nc [host] [port]      # send "data" then close
cat [file] | nc [host] [port]       # send file contents
```

### How to use OpenSSL

The `openssl` command is a toolkit for SSL/TLS and cryptographic operations. It can encrypt/decrypt, generate keys, inspect certificates, and connect to TLS-enabled services.

```bash
openssl [subcommand] [options]
```

Common subcommands:

```bash
openssl version                       # show version
openssl enc -aes-256-cbc -in file     # encrypt a file
openssl rand -hex 16                  # generate random bytes
openssl x509 -in cert.pem -text       # inspect a certificate
```

### How to connect to a TLS/SSL service with `openssl s_client`

The `s_client` subcommand of `openssl` opens a TLS/SSL connection to a server, similar to `telnet` but over an encrypted channel. It is used when a service requires TLS instead of plain TCP.

```bash
openssl s_client -connect [host]:[port]
```

Useful options:

```bash
openssl s_client -connect [host]:[port] -quiet       # suppress certificate info
openssl s_client -connect [host]:[port] -ign_eof     # don't exit on EOF
openssl s_client -connect [host]:[port] -showcerts   # show full cert chain
```

After the TLS handshake completes, the connection behaves like an interactive shell to the service — anything typed is sent encrypted to the server.

### How to scan ports with `nmap`

The `nmap` (Network Mapper) command scans hosts to discover open ports, running services, and their versions. It is the standard tool for network reconnaissance.

```bash
nmap [options] [target]
```

Common usage:

```bash
nmap [host]                        # default scan of top 1000 TCP ports
nmap -p [port] [host]              # scan a specific port
nmap -p [start]-[end] [host]       # scan a port range
nmap -sV [host]                    # detect service versions
nmap -A [host]                     # aggressive scan (OS, version, scripts)
nmap -sU [host]                    # UDP scan
nmap -Pn [host]                    # skip host discovery (assume up)
```

Example:

```bash
nmap -p 30000 localhost            # check if port 30000 is open on localhost
nmap -sV -p 30000 localhost        # also detect what service is running
```

---

## Steps

### 1. Log in with SSH as bandit14

```bash
ssh -p 2220 bandit14@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 13 → 14.

### 2. Verify that port 30000 is open on localhost

Use `nmap` to confirm the service is listening on port 30000.

```bash
nmap -p 30000 localhost
```

Expected output:

```
PORT      STATE SERVICE
30000/tcp open  ndmps
```

- `STATE: open` — the port is accepting connections over TCP.
- `SERVICE: ndmps` — this is just a guess based on the port number, not what's actually running.

### 3. Connect to the service and submit the bandit14 password

Connect to port 30000 over plain TCP using `telnet` (or `nc`), then send the password of the current level.

```bash
telnet localhost 30000
```

After the connection is established, type (or paste) the bandit14 password and press Enter.

Alternative one-liner using `nc`:

```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

This pipes the current password file directly into the service.

### 4. Receive the bandit15 password

The service responds with:

```
Correct!
<PASSWORD_FOR_BANDIT15>
```

The string returned after `Correct!` is the password for `bandit15`.

---

## Reference

- [Bandit Level 15 Page](https://overthewire.org/wargames/bandit/bandit15.html)