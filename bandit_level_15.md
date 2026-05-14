# Bandit Level 15 → Level 16

## Goal

The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL/TLS encryption.

**Helpful note:** Getting "DONE", "RENEGOTIATING" or "KEYUPDATE"? Read the "CONNECTED COMMANDS" section in the manpage.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit15`
- **Password:** the password obtained from Level 14 → 15

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

The `ssh` command opens a secure shell connection to a remote machine.

```bash
ssh [user]@[host] -p [port]
```

### How to connect to a TCP service with telnet

The `telnet` command opens an interactive plain-text TCP connection.

```bash
telnet [host] [port]
```

### How to use netcat (`nc`)

The `nc` command reads and writes data over plain TCP/UDP connections.

```bash
nc [host] [port]                  # connect to a TCP service
echo "data" | nc [host] [port]    # send data via pipe
```

`nc` does **not** speak TLS, so it cannot connect to TLS-only services.

### How to use `ncat`

`ncat` is an improved version of `nc` from the Nmap project. Unlike plain `nc`, it supports TLS directly with the `--ssl` option.

```bash
ncat [host] [port]                # plain TCP
ncat --ssl [host] [port]          # TLS connection
```

This makes `ncat` a quick alternative to `openssl s_client` for simple TLS interactions.

### How to use `socat`

`socat` (SOcket CAT) is a flexible relay tool that connects two data streams. It supports TCP, UDP, TLS, files, pipes, and more, in any combination.

```bash
socat - OPENSSL:[host]:[port],verify=0
```

- `-` — standard input/output as one endpoint
- `OPENSSL:host:port` — TLS connection as the other endpoint
- `verify=0` — skip certificate verification (useful for self-signed certs)

`socat` is often used when `openssl s_client` behavior is awkward (e.g., it sends extra `R` commands as renegotiation triggers).

### How to use OpenSSL

The `openssl` command is a toolkit for SSL/TLS and cryptographic operations.

```bash
openssl [subcommand] [options]
```

### How to connect to a TLS/SSL service with `openssl s_client`

The `s_client` subcommand opens a TLS connection to a server. It is the main way to manually interact with TLS services.

```bash
openssl s_client -connect [host]:[port]
```

Useful options:

```bash
openssl s_client -connect [host]:[port] -quiet       # suppress certificate output
openssl s_client -connect [host]:[port] -ign_eof     # don't exit on EOF
openssl s_client -connect [host]:[port] -showcerts   # show full certificate chain
openssl s_client -connect [host]:[port] -crlf        # convert LF to CRLF on input
```

**Important — CONNECTED COMMANDS:**

While connected through `s_client`, certain single-letter inputs are intercepted as **control commands** rather than being sent to the server:

|Input|Meaning|
|---|---|
|`Q`|Close the connection|
|`R`|Trigger renegotiation (causes "RENEGOTIATING")|
|`B`|Send a heartbeat|
|`K`|Trigger TLS 1.3 key update (causes "KEYUPDATE")|

If your input happens to contain these letters on a line by themselves, `s_client` will not send them — it interprets them as commands. This is why pasting a password might unexpectedly trigger "RENEGOTIATING" or "KEYUPDATE".

**Workarounds:**

```bash
openssl s_client -connect [host]:[port] -quiet -ign_eof   # disables some control parsing
openssl s_client -connect [host]:[port] -nocommands       # disable all control commands
```

`-nocommands` is the cleanest way to send any input safely without interference.

### How to scan ports with `nmap`

```bash
nmap [host]                       # default scan of top 1000 TCP ports
nmap -p [port] [host]             # scan a specific port
nmap -sV -p [port] [host]         # detect service version (and TLS usage)
```

`-sV` is especially useful for distinguishing plain TCP from TLS services — TLS services show up with an `ssl/` prefix in the output.

### How to check listening ports with `netstat`

The `netstat` (NETwork STATistics) command shows network connections, listening ports, routing tables, and more. On modern systems it has been mostly replaced by `ss`, but it's still common.

```bash
netstat -tlnp           # TCP listening sockets with process info
netstat -ulnp           # UDP listening sockets with process info
netstat -an             # all connections, numeric (no DNS resolution)
```

Option breakdown:

- `-t` — TCP
- `-u` — UDP
- `-l` — listening only
- `-n` — show numbers, not names
- `-p` — show process info (may need sudo)
- `-a` — all sockets

### How to check listening ports with `ss`

`ss` (Socket Statistics) is the modern replacement for `netstat`. It's faster and pulls info directly from the kernel.

```bash
ss -tlnp                # TCP listening sockets with process info
ss -ulnp                # UDP listening sockets with process info
ss -tan                 # all TCP connections, numeric
ss -tan | grep LISTEN   # filter only listening sockets
```

The options mirror `netstat` for the most part, making the transition easy.

---

## Steps

### 1. Log in with SSH as bandit15

```bash
ssh -p 2220 bandit15@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 14 → 15.

### 2. (Optional) Verify the service on port 30001

Check that port 30001 is open and confirm it speaks TLS:

```bash
nmap -p 30001 localhost
nmap -sV -p 30001 localhost
```

The `-sV` output shows an `ssl/` prefix on the service, indicating the port requires TLS (not plain TCP). This is why `nc` or `telnet` will not work here.

### 3. Connect to the TLS service with `openssl s_client`

```bash
openssl s_client -connect localhost:30001
```

The output shows the TLS handshake details: the server certificate (a self-signed `SnakeOil` cert), the negotiated cipher (TLSv1.3, `TLS_AES_256_GCM_SHA384`), and a session ticket. A self-signed certificate produces a `verify error: self-signed certificate` warning, which is safe to ignore for this exercise.

After the handshake, the connection enters interactive mode (`read R BLOCK`) and waits for input.

### 4. Send the bandit15 password

Paste (or type) the bandit15 password and press Enter.

The service verifies the password and responds with:

```
Correct!
<PASSWORD_FOR_BANDIT16>
```

The returned string is the password for `bandit16`.

### 5. Exit the connection

Press `Ctrl+C` or close the terminal to disconnect.

---

## Notes

- The behavior is essentially the same as Level 14 → 15, except the channel is encrypted with TLS. The application-level protocol is identical: send password, receive the next password.
- If a single-letter line in the input accidentally triggers `RENEGOTIATING` or `KEYUPDATE`, use `-nocommands` to disable that interpretation.

---

## Reference

- [Bandit Level 16 Page](https://overthewire.org/wargames/bandit/bandit16.html)
- [TLS on Wikipedia](https://en.wikipedia.org/wiki/Transport_Layer_Security)
- [OpenSSL Cookbook - Testing with OpenSSL](https://www.feistyduck.com/library/openssl-cookbook/online/testing-with-openssl/index.html)
