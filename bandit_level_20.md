# Bandit Level 20 → Level 21

## Goal

There is a setuid binary in the home directory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

**NOTE:** Try connecting to your own network daemon to see if it works as you think.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit20`
- **Password:** the password obtained from Level 19 → 20

---

## Hint

- `ssh`
- `nc`
- `cat`
- `bash`
- `screen`
- `tmux`
- Unix 'job control' (`bg`, `fg`, `jobs`, `&`, `CTRL-Z`, ...)

---

## Solution

### How to log in with SSH

```bash
ssh [user]@[host] -p [port]
```

### How to view a file's contents

```bash
cat [file name]
```

### How to use netcat (`nc`) as a server (listener)

The `nc` command can act as both a client and a **server** (listener). For this level, the listener role is essential.

**Client mode (connect to a service):**

```bash
nc [host] [port]
echo "data" | nc [host] [port]
```

**Server mode (listen on a port):**

```bash
nc -l -p [port]            # listen on a port
nc -lvp [port]             # listen + verbose
```

When in listening mode, `nc` waits for an incoming connection. Once a client connects, `nc` reads data from the client and outputs anything you type (or pipe in) back to the client.

**Listening mode with predefined response:**

```bash
echo "data to send" | nc -l -p [port]
```

This makes `nc` listen for a connection, and when a client connects, it immediately sends `data to send` to that client.

### Why a listener matters in this level

The setuid binary acts as a **client**: it connects to a port on localhost and expects to receive the bandit20 password on that connection. So the player must run a server (listener) on some port that sends the bandit20 password back, then run the binary pointing at that port.

```
[setuid binary]  →  connects to localhost:PORT  →  [your nc listener]
                                                   sends bandit20 password
                                                                ↓
[setuid binary]  ←  receives password, verifies, sends back bandit21 password  ←
```

### Choosing a port number

Since the player is creating the listener, the port number is chosen freely:

- **0–1023**: Privileged ports (root only) — avoid
- **1024–49151**: Generally usable
- **49152–65535**: Usually unused

For learning, common choices are `9999`, `8888`, `12345`, `31337`. Any unused port in the available range works.

### How to use `bash` job control

The challenge requires running **two things at the same time** in a single SSH session: a listener and the setuid binary. Job control lets you manage multiple processes in one shell.

**Run a command in the background:**

```bash
[command] &
```

The `&` at the end starts the command and immediately returns the shell prompt. The job runs in the background while the next command can be entered.

**Other job control essentials:**

|Command|Meaning|
|---|---|
|`jobs`|List background jobs in the current shell|
|`fg %N`|Bring job N to the foreground|
|`bg %N`|Resume job N in the background|
|`CTRL-Z`|Suspend the currently running foreground job|
|`kill %N`|Terminate job N|

### How to use `screen` and `tmux` (alternative to job control)

`screen` and `tmux` are **terminal multiplexers** — they let you run multiple shells inside a single SSH session, each in its own "window" or "pane." They are an alternative to backgrounding jobs.

**`tmux` basics:**

```bash
tmux                    # start a new tmux session
CTRL-b "                # split window horizontally
CTRL-b %                # split window vertically
CTRL-b [arrow]          # move between panes
CTRL-b d                # detach (leave running in background)
tmux attach             # reattach to a detached session
```

**`screen` basics:**

```bash
screen                  # start a new screen session
CTRL-a c                # create a new window
CTRL-a n                # switch to next window
CTRL-a d                # detach
screen -r               # reattach
```

For this level, simple job control with `&` is sufficient. Multiplexers shine when managing multiple long-running tasks.

### How to run the setuid binary

The challenge says the binary is in the home directory. List the home directory contents and run it with `./` prefix:

```bash
ls
./suconnect            # without arguments, prints usage
./suconnect [port]     # connect to localhost on this port
```

---

## Steps

### 1. Log in as bandit20

```bash
ssh -p 2220 bandit20@bandit.labs.overthewire.org
```

When prompted, enter the password from Level 19 → 20.

### 2. Identify the setuid binary

```bash
ls -l
```

A file like `suconnect` should be visible, with `s` in the owner's execute slot, indicating setuid:

```
-rwsr-x--- 1 bandit21 bandit20 ... suconnect
```

### 3. Check usage by running without arguments

```bash
./suconnect
```

Output:

```
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP.
If it receives the correct password from the other side, the next password is transmitted back.
```

This confirms the workflow: pick a port, run a listener on it that sends the bandit20 password, then run `./suconnect [port]`.

### 4. Get the bandit20 password ready

The password is needed to feed into the listener. It can be read from `/etc/bandit_pass/bandit20` (readable by bandit20) or pasted manually.

```bash
cat /etc/bandit_pass/bandit20
```

### 5. Start a listener on a chosen port, in the background

Pick a port (e.g., `9999`) and start `nc` in listening mode. Use `echo` (or `cat`) to pipe the bandit20 password into the listener, and append `&` to run it in the background.

```bash
echo "<BANDIT20_PASSWORD>" | nc -l -p 9999 &
```

The shell prints something like `[1] 1234` (job number and PID), then returns the prompt. The listener is now waiting for a connection on port 9999, and will reply with the password as soon as something connects.

### 6. Run the setuid binary against the listener

```bash
./suconnect 9999
```

What happens:

1. `suconnect` connects to `localhost:9999`
2. The listener (background `nc`) sends the bandit20 password
3. `suconnect` verifies the password matches
4. `suconnect` sends back the bandit21 password through the same connection
5. The background `nc` prints that reply to the terminal before exiting

Example output:

```
Read: <BANDIT20_PASSWORD>
Password matches, sending next password
<BANDIT21_PASSWORD>
[1]+  Done                    echo "..." | nc -l -p 9999
```

The line after "sending next password" is the password for `bandit21`.

---

## Notes

- This is the first level where the player runs **a service** instead of only connecting to one. The mental shift — from client to server — is significant in security work.
- The `[command] &` pattern is the simplest form of concurrency in a shell. Real automation pipelines, attack tooling, and post-exploitation scripts use the same primitive.
- In real penetration testing, the same pattern appears in **reverse shells**: the attacker runs `nc -lvp [port]` to receive an incoming connection from a compromised target. The roles of "server" and "client" are swapped from what most people expect.
- If `&` is forgotten and the listener blocks the shell, press `CTRL-Z` to suspend it, then run `bg` to move it to the background — this also works.

---

## Reference

- [Bandit Level 21 Page](https://overthewire.org/wargames/bandit/bandit21.html)
- `man nc` — listening mode
- `man bash` — JOB CONTROL section
- `man tmux`, `man screen`