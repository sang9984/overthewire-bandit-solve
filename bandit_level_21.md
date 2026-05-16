# Bandit Level 21 → Level 22

## Goal

A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

---

## Given Info

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Username:** `bandit21`
- **Password:** the password obtained from Level 20 → 21

---

## Hint

- `cron`
- `crontab`
- `crontab(5)` (use `man 5 crontab` to access this)

---

## Solution

### What is `cron`

`cron` is a time-based job scheduler in Unix-like systems. It runs commands automatically at specified intervals (every minute, every hour, every day, every week, etc.) without user intervention.

In security context, cron is a major **privilege escalation surface**: if a cron job runs as a privileged user and executes a script (or writes to a file) that a lower-privileged user can read or modify, that user can gain information or escalate privileges.

### `cron` configuration locations

|Path|Purpose|
|---|---|
|`/etc/crontab`|System-wide cron table|
|`/etc/cron.d/`|Additional system cron jobs (one file per job)|
|`/etc/cron.hourly/`|Scripts run hourly|
|`/etc/cron.daily/`|Scripts run daily|
|`/etc/cron.weekly/`|Scripts run weekly|
|`/etc/cron.monthly/`|Scripts run monthly|
|`/var/spool/cron/crontabs/`|Per-user crontabs|

For this level, the challenge points to `/etc/cron.d/`.

### `crontab(5)` syntax

A system crontab entry has seven fields:

```
* * * * * user command
│ │ │ │ │
│ │ │ │ └─ Day of week (0-7, 0 and 7 = Sunday)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

Special characters:

- `*` — every value (matches anything in that field)
- `*/N` — every N units (e.g., `*/30` = every 30 minutes)
- `1,3,5` — specific values
- `1-5` — range

Special shortcuts:

- `@reboot` — once at system boot
- `@daily` — same as `0 0 * * *`
- `@hourly` — same as `0 * * * *`

### Shell wildcards (globbing)

The `*` character in shell commands expands to **all matching files** in the current path:

```bash
cat /etc/cron.d/*
```

The shell expands this before running `cat`, so all files in `/etc/cron.d/` are passed as arguments. This is useful for inspecting multiple files at once.

### Redirect operators

|Operator|Meaning|
|---|---|
|`>`|Redirect output to file (overwrite)|
|`>>`|Redirect output to file (append)|
|`&>`|Redirect both stdout and stderr|
|`2>&1`|Merge stderr into stdout|
|`> /dev/null`|Discard output (send to "black hole")|

Example:

```bash
cat /etc/bandit_pass/bandit22 > /tmp/somefile
# Reads bandit22 password file and writes it to /tmp/somefile
```

### How to check file permissions

```bash
ls -l [file]      # Show permissions, owner, group
ls -la [file]     # Same, including hidden files
```

### How to view file contents

```bash
cat [file]
```

### Tab completion

Long, randomly-named files are easy to mistype. Use Tab autocomplete in the shell:

```bash
cat /tmp/t7  [Tab]
# Shell completes the filename automatically
```

### Workflow for this level

1. Log in as bandit21
2. List `/etc/cron.d/` contents
3. Read all cron files to see which jobs run as which users
4. Identify the cron job that runs as bandit22
5. Read the script that the cron job executes
6. Understand what the script does (writes bandit22 password to a file)
7. Read that target file to obtain the password

---

## Steps

### 1. Log in as bandit21

```bash
ssh -p 2220 bandit21@bandit.labs.overthewire.org
```

Enter the password from Level 20 → 21.

### 2. List the cron configuration directory

```bash
ls /etc/cron.d/
```

Example output:

```
behemoth4_cleanup  clean_tmp  cronjob_bandit22  cronjob_bandit23  cronjob_bandit24
e2scrub_all  leviathan5_cleanup  manpage3_resetpw_job  otw-tmp-dir  sysstat
```

Multiple cron files are present. The filenames hint that some are related to bandit22/23/24, but this should be verified by reading the file contents (file names are not a guarantee of behavior).

### 3. Read all cron files

```bash
cat /etc/cron.d/*
```

This uses shell globbing to read every file in the directory at once. Some files may show `Permission denied` (other wargames' configurations the player has no access to) — these can be ignored.

Look for cron lines that run as `bandit22`:

```
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

Interpretation:

- `@reboot` — run once at system boot, as bandit22
- `* * * * *` — run every minute, as bandit22
- Both execute the same script: `/usr/bin/cronjob_bandit22.sh`
- `&> /dev/null` — discard all output

### 4. Read the cron script

```bash
cat /usr/bin/cronjob_bandit22.sh
```

Output:

```bash
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Analysis:

- Line 1: `chmod 644 /tmp/...` — makes the target file world-readable
- Line 2: `cat /etc/bandit_pass/bandit22 > /tmp/...` — reads bandit22's password file and writes it to `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv`

Since this script runs every minute as bandit22, the password file at `/tmp/...` is continuously refreshed and readable by anyone (mode 644).

### 5. Read the password file

```bash
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

The output is the password for `bandit22`.

> **Tip:** The filename is long and random — use Tab autocomplete rather than typing it manually:
> 
> ```bash
> cat /tmp/t7O6  [Tab]
> ```

---

## Notes

- This level demonstrates a classic **privilege information leak via scheduled tasks**. A higher-privileged user's secret is exposed because an automated task writes it to a world-readable location.
- In real-world security, this pattern appears as:
    - Cron jobs writing sensitive data to `/tmp` or logs
    - Backup scripts copying credentials to readable backup locations
    - Debug scripts left enabled in production
- The fix would be: never write secrets to world-readable paths, run scripts with the minimum necessary privileges, and audit cron jobs regularly.
- The technique used here is **information disclosure via cron**. It is not privilege escalation per se (the player doesn't gain bandit22's shell), but it grants access to bandit22's credentials, which then enables logging in as bandit22.

---

## Reference

- [Bandit Level 22 Page](https://overthewire.org/wargames/bandit/bandit22.html)
- `man 5 crontab` — crontab file format
- `man cron` — cron daemon
- `man bash` — see "REDIRECTION" section for `>`, `>>`, `&>`