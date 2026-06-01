# Gobuster: The Basics

> TryHackMe | Cyber Security 101  
> Room: [Gobuster: The Basics](https://tryhackme.com/room/gobusterthebasics)  
> Difficulty: Easy | Tags: Reconnaissance, Enumeration, Web, DNS, Brute Force

---

## 📌 Overview

Gobuster is an open-source offensive tool written in Go, commonly used during
the reconnaissance and scanning phases of a penetration test. It enumerates
web directories, DNS subdomains, virtual hosts, Amazon S3 buckets, and Google
Cloud Storage by brute force using wordlists.

**Topics covered:**
- What enumeration and brute force mean in context
- Gobuster global flags and how to use them
- `dir` mode — enumerate web directories and files
- `dns` mode — enumerate DNS subdomains
- `vhost` mode — enumerate virtual hosts
- How to interpret results and filter false positives

**Lab environment:**
- Target: Ubuntu 20.04 web server at `10.49.189.70`
- Hosts multiple subdomains, vhosts, WordPress, and Joomla installs
- AttackBox has Gobuster pre-installed

---

## 🔧 Task 2 — Environment Setup

Before starting, configure the AttackBox to resolve the lab's custom domains
using the web server's local DNS.

### Step 1 — Add the nameserver

```bash
sudo nano /etc/resolv-dnsmasq
```

Insert as the **first line**:

```
nameserver 10.49.189.70
```

The file should look like:

```
nameserver 10.49.189.70
nameserver 169.254.169.253
```

Save: `CTRL+O` → `Enter` → Exit: `CTRL+X`

### Step 2 — Restart Dnsmasq

```bash
/etc/init.d/dnsmasq restart
```

> ⚠️ This step is required. Without it, Gobuster's DNS and vhost modes
> won't resolve the lab domains, and all scans will fail silently or return
> incorrect results.

---

## 🧠 Task 3 — Gobuster Overview

### Core Concepts

**Enumeration** — listing all available resources, whether accessible or not.
Gobuster enumerates web directories, subdomains, and virtual hosts.

**Brute Force** — trying every possibility until a match is found. Gobuster
uses wordlists to generate each candidate and sends a request for each one.

### Gobuster Modes

```bash
gobuster --help
```

| Command | What it does |
|---------|-------------|
| `dir` | Directory and file enumeration mode |
| `dns` | DNS subdomain enumeration mode |
| `vhost` | Virtual host enumeration mode |
| `fuzz` | Fuzzing mode — replaces `FUZZ` keyword in URL, headers, or body |
| `s3` | AWS S3 bucket enumeration |
| `gcs` | Google Cloud Storage bucket enumeration |
| `tftp` | TFTP enumeration |

### Global Flags (used across all modes)

| Short | Long | Description |
|-------|------|-------------|
| `-t` | `--threads` | Number of concurrent threads (default: 10). Increase for faster scans on large wordlists. |
| `-w` | `--wordlist` | Path to the wordlist file. Each entry is tested against the target. |
| | `--delay` | Time to wait between requests (e.g. `1500ms`). Useful to evade rate-limiting detection. |
| | `--debug` | Enable debug output for troubleshooting unexpected errors. |
| `-o` | `--output` | Write results to a file instead of stdout. |
| `-v` | `--verbose` | Verbose output — shows errors as well as findings. |
| `-q` | `--quiet` | Suppress banner and noise — cleaner output. |
| `-z` | `--no-progress` | Don't display progress bar. |
| | `--no-color` | Disable color output (useful when piping to a file). |
| | `--no-error` | Don't display errors. |

### Basic Example

```bash
gobuster dir -u "http://www.example.thm/" -w /usr/share/wordlists/dirb/small.txt -t 64
```

Breaking it down:
- `dir` — use directory enumeration mode
- `-u "http://www.example.thm/"` — target URL (protocol required)
- `-w .../small.txt` — wordlist to brute force with
- `-t 64` — use 64 threads (default 10 is slow for large wordlists)

**Task answers:**
- Flag to specify target URL → **`-u`**
- Command for subdomain enumeration → **`dns`**

---

## 🗂️ Task 4 — dir Mode: Directory and File Enumeration

### What it does

`dir` mode sends GET requests to the target URL with each wordlist entry
appended as a path. The HTTP response code tells you whether the path exists
and is accessible.

```
http://target.thm/ + "images" → GET http://target.thm/images/
http://target.thm/ + "admin"  → GET http://target.thm/admin/
```

### dir Mode Flags

| Short | Long | Description |
|-------|------|-------------|
| `-u` | `--url` | Target URL **(required)** |
| `-w` | `--wordlist` | Wordlist path **(required)** |
| `-x` | `--extensions` | File extensions to scan for (e.g. `.php,.js,.txt`) |
| `-r` | `--followredirect` | Follow HTTP redirects (301, 302) |
| `-s` | `--status-codes` | Only show results with these status codes |
| `-b` | `--status-codes-blacklist` | Hide results with these status codes (overrides `-s`) |
| `-n` | `--no-status` | Don't show status codes in output |
| `-k` | `--no-tls-validation` | Skip TLS certificate check (for self-signed certs — common in CTFs) |
| `-c` | `--cookies` | Pass a cookie with each request (e.g. session ID for authenticated scans) |
| `-H` | `--headers` | Add a custom header to each request |
| `-U` | `--username` | Username for authenticated requests (use with `-P`) |
| `-P` | `--password` | Password for authenticated requests (use with `-U`) |

> 💡 `-b` overrides `-s`. If you set both, `-b` takes precedence.

### Important Behavior Notes

- **Gobuster does not enumerate recursively.** If it finds `/secret/`, you
  must run a second scan targeting `http://target.thm/secret/` to enumerate
  its contents.
- The `-u` value sets the base path. You can target a subdirectory directly:
  `-u "http://target.thm/resources/"` scans inside `/resources/`.
- Use **hostname** (not IP) when the server uses virtual hosting — using the
  IP may scan the wrong site.
- The URL **must include the protocol** (`http://` or `https://`). Without
  it, the scan fails.

---

### 🔍 Practical: Directory Enumeration of offensivetools.thm

#### Step 1 — Add the host to `/etc/hosts`

Since there is no public DNS for `offensivetools.thm`, add it manually:

```bash
sudo nano /etc/hosts
```

Add this line (replace with your target IP):

```
10.49.189.70    www.offensivetools.thm offensivetools.thm
```

Save: `CTRL+O` → `Enter` → Exit: `CTRL+X`

> 💡 Why `/etc/hosts`? The `/etc/hosts` file is checked before DNS. Adding
> the entry here tells your machine to resolve `www.offensivetools.thm`
> directly to the target IP without needing a DNS server.

#### Step 2 — Enumerate directories

```bash
gobuster dir -u "http://www.offensivetools.thm" \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -r
```

The `-r` flag follows redirects — useful since web servers often redirect
`/secret` to `/secret/` with a 301.

**Result that catches attention:** `/secret` (returns 200 — accessible)

#### Step 3 — Enumerate the `secret` directory for files

Run a second scan specifically targeting the discovered directory, and add
the `-x` flag to search for specific file extensions:

```bash
gobuster dir -u "http://www.offensivetools.thm/secret/" \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x .js
```

**Result:** `flag.js` found inside `/secret/`

#### Step 4 — Read the flag

```bash
curl http://www.offensivetools.thm/secret/flag.js
```

**Flag:** `THM{ReconWasASuccess}`

> 💡 `curl` is the fastest way to read a file's contents without opening a
> browser. It sends a GET request and prints the response body to the terminal.

**Task answers:**
- Flag to skip TLS verification → **`--no-tls-validation`**
- Directory that catches attention → **`secret`**
- Flag found in the `.js` file → **`THM{ReconWasASuccess}`**

---

## 🌐 Task 5 — dns Mode: Subdomain Enumeration

### What it does

`dns` mode performs DNS lookups. For each wordlist entry, Gobuster constructs
a subdomain FQDN and queries the configured DNS server for it.

```
wordlist entry: "www"   → DNS lookup: www.example.thm → resolved? → found
wordlist entry: "shop"  → DNS lookup: shop.example.thm → resolved? → found
wordlist entry: "abc123"→ DNS lookup: abc123.example.thm → NXDOMAIN → skip
```

> 💡 **Why enumerate subdomains?** A vulnerability patched on the main domain
> may still be present on a subdomain. `tryhackme.thm` may be fully patched
> while `mobile.tryhackme.thm` runs an older, vulnerable version of the same
> software.

### dns Mode Flags

| Short | Long | Description |
|-------|------|-------------|
| `-d` | `--domain` | Target domain **(required)** |
| `-w` | `--wordlist` | Wordlist path **(required)** |
| `-r` | `--resolver` | Custom DNS server to use for resolving (IP address) |
| `-c` | `--show-cname` | Show CNAME records (cannot be used with `-i`) |
| `-i` | `--show-ips` | Show IP addresses the subdomains resolve to |

### Basic syntax

```bash
gobuster dns -d example.thm -w /path/to/wordlist
```

### Example output

```
Found: www.example.thm
Found: shop.example.thm
Found: academy.example.thm
Found: primary.example.thm
```

---

### 🔍 Practical: Subdomain Enumeration of offensivetools.thm

```bash
gobuster dns -d offensivetools.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  -r 10.49.189.70
```

Flag breakdown:
- `-d offensivetools.thm` — target domain
- `-w .../subdomains-top1million-5000.txt` — wordlist of common subdomains
- `-r 10.49.189.70` — use the lab's local DNS server to resolve (required
  since `offensivetools.thm` is not in public DNS)

**Result:** 4 subdomains found

**Task answers:**
- Required shorthand flag besides `-w` → **`-d`**
- Number of subdomains configured → **`4`**

---

## 🖥️ Task 6 — vhost Mode: Virtual Host Enumeration

### What it does

`vhost` mode brute forces virtual hosts — multiple websites hosted on the
same IP address/server, differentiated by the `Host:` header in HTTP requests.

**Virtual hosts vs subdomains — the key difference:**

| | Virtual Hosts | Subdomains |
|-|--------------|------------|
| **Resolution** | IP-based — same server, different `Host:` header | DNS-based — different DNS records |
| **Discovery method** | `vhost` mode — sends HTTP requests with different `Host:` values | `dns` mode — performs DNS lookups |
| **Can exist without DNS?** | Yes — no DNS entry required | No — requires a DNS record |

**How Gobuster builds vhost requests:**

```
GET / HTTP/1.1
Host: www.example.thm       ← Gobuster changes this for each wordlist entry
User-Agent: gobuster/3.6
...
```

Breaking `www.example.thm` into parts:
- `www` — subdomain, filled in from the wordlist
- `.example` — second-level domain, set with `--domain`
- `.thm` — top-level domain, set with `--domain`

### vhost Mode Flags

| Short | Long | Description |
|-------|------|-------------|
| `-u` | `--url` | Base URL / target IP **(required)** |
| `-w` | `--wordlist` | Wordlist path **(required)** |
| | `--domain` | Appends the domain to each wordlist entry to form a valid hostname |
| | `--append-domain` | Appends the configured domain to each wordlist entry — **must use this to avoid false positives** |
| | `--exclude-length` | Exclude responses by body size — used to filter false positives |
| `-m` | `--method` | HTTP method to use (default: GET) |
| `-r` | `--follow-redirect` | Follow HTTP redirects |

### Why `--append-domain` matters

Without `--append-domain`:
```
Host: www         ← incorrect, will generate false positives
Host: blog        ← incorrect
```

With `--append-domain`:
```
Host: www.example.thm   ← correct
Host: blog.example.thm  ← correct
```

### Why `--exclude-length` matters

Without it, you get many false positives:
```
Found: Orion.example.thm     Status: 404 [Size: 279]
Found: pm.example.thm        Status: 404 [Size: 276]
```

False positives typically share similar response body sizes. By excluding
that size range, only legitimate vhosts surface in the results.

---

### 🔍 Practical: Virtual Host Enumeration of offensivetools.thm

```bash
gobuster vhost -u "http://10.49.189.70" \
  --domain offensivetools.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain \
  --exclude-length 250-320
```

Flag breakdown:
- `-u "http://10.49.189.70"` — scan via IP (we don't have full DNS setup)
- `--domain offensivetools.thm` — sets the domain appended to wordlist entries
- `--append-domain` — ensures entries become `word.offensivetools.thm`, not just `word`
- `--exclude-length 250-320` — filters out responses in this size range, which
  correspond to false positives (404 pages with similar body lengths)

**Result:** 4 virtual hosts responding with status 200

**Task answer:**
- vhosts replying with status 200 → **`4`**

---

## 📋 Full Command Reference

### dir mode

```bash
# Basic directory scan
gobuster dir -u "http://target.thm" -w /path/to/wordlist

# With file extension search
gobuster dir -u "http://target.thm" -w /path/to/wordlist -x .php,.js,.txt

# Follow redirects, skip TLS check (for HTTPS with self-signed cert)
gobuster dir -u "https://target.thm" -w /path/to/wordlist -r -k

# Authenticated scan
gobuster dir -u "http://target.thm" -w /path/to/wordlist -U admin -P password

# Save output to file, 64 threads
gobuster dir -u "http://target.thm" -w /path/to/wordlist -t 64 -o results.txt

# Scan a subdirectory
gobuster dir -u "http://target.thm/secret/" -w /path/to/wordlist -x .js
```

### dns mode

```bash
# Basic subdomain enumeration
gobuster dns -d example.thm -w /path/to/wordlist

# Use custom DNS resolver + show IPs
gobuster dns -d example.thm -w /path/to/wordlist -r 10.49.189.70 -i

# Show CNAME records
gobuster dns -d example.thm -w /path/to/wordlist -c
```

### vhost mode

```bash
# Basic vhost enumeration
gobuster vhost -u "http://target.thm" -w /path/to/wordlist

# Full flags for lab environments without proper DNS
gobuster vhost -u "http://10.49.189.70" \
  --domain example.thm \
  -w /path/to/wordlist \
  --append-domain \
  --exclude-length 250-320
```

### Read discovered file content

```bash
curl http://target.thm/path/to/file.js
```

---

## 🔍 Thinking Through the Room — Decision Flow

```
What do I want to enumerate?
│
├── Directories / files on a web server?
│   └── gobuster dir
│       ├── Required: -u (URL) + -w (wordlist)
│       ├── Found interesting dir? → scan it again with new -u
│       └── Looking for specific files? → add -x .php,.js
│
├── Subdomains (DNS-based)?
│   └── gobuster dns
│       ├── Required: -d (domain) + -w (wordlist)
│       └── Lab DNS? → add -r <DNS_SERVER_IP>
│
└── Virtual hosts (same IP, different Host: header)?
    └── gobuster vhost
        ├── Required: -u (URL) + -w (wordlist)
        ├── No full DNS setup? → add --domain + --append-domain
        └── Too many false positives? → add --exclude-length <range>
```

---

## 💡 Insights & Things to Remember

**Gobuster does not recurse automatically:**  
If you find `/secret/` in a scan, that's your cue to run a second scan with
`-u "http://target.thm/secret/"`. Every interesting directory is a new
entry point that needs its own scan. This is why taking notes during
enumeration matters.

**`--append-domain` is not optional in vhost mode without full DNS:**  
Forgetting it makes every wordlist entry a false positive. The `Host:` header
becomes just `www` instead of `www.example.thm` — the server doesn't know
what to do with it and returns a generic response for every single entry.

**vhost ≠ subdomain — the tool choice matters:**  
A virtual host can exist with no DNS record at all — it only needs a server
configured to respond to that `Host:` header. `dns` mode will never find it
because there is nothing to look up. Only `vhost` mode sends HTTP requests
and checks the response.

**`curl` is the fastest way to read a found file:**  
Once Gobuster finds `flag.js`, you don't need to open a browser. `curl URL`
prints the raw response in seconds.

**`--exclude-length` requires trial and error:**  
Run vhost without it first, note the sizes of obvious false positive 404
responses, then add `--exclude-length` with that range. This is a skill that
improves with practice.

---

## ⚖️ Reflection

**What clicked:**
- The DNS vs vhost distinction finally makes sense when you see *how* each
  mode works — dns does a lookup, vhost sends an HTTP request. They solve
  different problems even though the output looks similar
- The `--append-domain` flag explanation via the raw HTTP request breakdown
  is the clearest explanation I've seen — seeing the actual `Host:` header
  makes it obvious why the flag is necessary
- Two-step enumeration (find directory → enumerate directory) is the correct
  mental model for `dir` mode, not a limitation

**Still unclear:**
- When would I use `fuzz` mode vs `dir` mode? Fuzzing replaces `FUZZ`
  anywhere in the request — headers, body, URL parameters. Worth exploring
  for parameter fuzzing use cases.
- What wordlist is best for which scenario? `dirbuster` lists for dirs,
  `SecLists/DNS` for subdomains — but is there a systematic way to choose?

**Next:**
- SQL Injection room — apply web reconnaissance skills to find and exploit
  vulnerable endpoints that Gobuster helped locate
- Burp Suite room — pair with Gobuster for manual verification of found paths

---

## 📚 References

- [TryHackMe - Gobuster: The Basics](https://tryhackme.com/room/gobusterthebasics)
- [Gobuster GitHub Repository](https://github.com/OJ/gobuster)
- [SecLists Wordlists](https://github.com/danielmiessler/SecLists)
- [DirBuster Wordlists](https://gitlab.com/kalilinux/packages/dirbuster)
