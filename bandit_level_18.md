# Bandit Level 18 → Level 19

## Goal

The password for the next level is stored in a file `readme` in the home directory. Unfortunately, someone has modified `.bashrc` to log you out when you log in with SSH.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit18`
- **Password:** the password obtained from Level 17 → 18

---

## Hint

- `ssh`
- `ls`
- `cat`

---

## Solution

### Why the normal login fails

When you SSH into a Linux server, several startup files run automatically:

- `/etc/profile`, `/etc/bashrc` — system-wide startup
- `~/.bash_profile`, `~/.bashrc`, `~/.profile` — user-level startup

These files are sourced **as soon as the shell starts**, before the user can type anything. In this level, bandit18's `.bashrc` has been modified to print `Byebye!` and immediately exit the shell. So even though authentication succeeds, the shell terminates before any command can be run interactively.

### How `ssh` can run a command remotely

The `ssh` command isn't limited to opening an interactive shell. You can pass a command as an extra argument, and SSH will:

1. Connect to the remote host
2. Authenticate
3. Execute that single command
4. Return the output to your local terminal
5. Disconnect

**Syntax:**

```bash
ssh [user]@[host] -p [port] "[remote command]"
```

The command runs in a **non-interactive shell**, which behaves slightly differently from an interactive shell. In particular, depending on the shell configuration, some startup files (like `.bashrc`'s interactive sections) may be skipped — allowing the command to execute even when interactive login is blocked.

**Examples:**

```bash
ssh user@host "ls"                   # list files in remote home directory
ssh user@host "cat /etc/hostname"    # read a remote file
ssh user@host "whoami"               # check the remote user
```

This trick lets you bypass shells that are configured to exit immediately, as long as your command runs before (or instead of) the exit logic.

### Combining `cat` with remote execution

To read the `readme` file in bandit18's home directory without entering an interactive shell:

```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org "cat readme"
```

After entering the bandit18 password, the remote `cat readme` runs and prints the file contents back to your terminal. The shell never enters interactive mode, so the `Byebye!` exit logic never gets a chance to kick in.

---

## Steps

### 1. Confirm that interactive login fails

```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org
```

After entering the password, the server responds with `Byebye!` and immediately closes the connection. This confirms that **authentication is successful**, but the shell exits before any command can be typed.

### 2. Use `ssh` with a remote command

Run the desired command directly through SSH, without opening an interactive session:

```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org "cat readme"
```

- `ssh -p 2220 bandit18@bandit.labs.overthewire.org` — connect as bandit18
- `"cat readme"` — the command to execute on the remote side

### 3. Authenticate

Enter the bandit18 password when prompted. SSH runs the `cat readme` command on the remote system before the interactive shell logic (and the `Byebye!` exit) takes over.

### 4. Read the output

The contents of `readme` are printed to your local terminal. This contains the password for `bandit19`.

```
<PASSWORD_FOR_BANDIT19>
```

---

## Notes

- This level illustrates an important distinction: **authentication, interactive shells, and remote command execution are separate things**. Just because an interactive shell exits, it doesn't mean you can't run commands on the system.
- In real-world security work, this pattern (`ssh user@host "command"`) is commonly used for automation, configuration management (Ansible, Fabric), and quick diagnostics.
- The `Byebye!` trick is a simple example of a restricted shell or login banner. Real-world systems may use `chsh` to set `/sbin/nologin` as the user's shell, or use `ForceCommand` in `sshd_config` to enforce a fixed command — these are more robust restrictions that can't be bypassed the same way.
- For more complex restrictions, options like `ssh -t` (force pseudo-terminal allocation) or running specific commands like `bash --noprofile --norc` may also be useful.

---

## Reference

- [Bandit Level 19 Page](https://overthewire.org/wargames/bandit/bandit19.html)
- `man ssh` — see the "Command execution" section
- `man bash` — see the "INVOCATION" section for how startup files differ between interactive and non-interactive shells