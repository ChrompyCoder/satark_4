# CyberSec Beginner Lab Guide

## Objective

Hands-on practical where participants scan a website, discover a hidden password-protected ZIP file, and crack it using John the Ripper.

---

## Tools Required

| Tool | Purpose | Install |
|---|---|---|
| **Gobuster** | Directory & file scanning | `apt install gobuster` \| `brew install gobuster` \| [GitHub releases](https://github.com/OJ/gobuster/releases) |
| **curl** | Manual recon (robots.txt, page source) | Pre-installed on most systems |
| **wget** | Download target files | `apt install wget` \| `brew install wget` |
| **John the Ripper** | Password cracking | `apt install john` \| `brew install john` \| [openwall.com/john](https://www.openwall.com/john/) |
| **zip2john** | Extract hash from ZIP (included with John) | Bundled with John the Ripper |

---

## Part 1: Instructor Setup

### 1.1 Create the Protected ZIP

```bash
# Create a flag file
echo "FLAG{y0u_cr4ck3d_1t}" > flag.txt

# Create a password-protected ZIP (use a weak password intentionally)
zip -P s3cure123 challenge.zip flag.txt

# Clean up the plaintext
rm flag.txt
```

### 1.2 Host on GitHub Pages

Structure your `username.github.io` repo like this:

```
your-username.github.io/
├── index.html          ← landing page with hints
├── challenge.zip       ← the protected ZIP
├── /backup/
│   └── secret.zip      ← hidden file for scanning exercise
└── robots.txt          ← intentional clue for recon
```

**robots.txt** — deliberately leak a path:

```
User-agent: *
Disallow: /backup/
```

**index.html** — simple landing page:

```html
<!DOCTYPE html>
<html>
<head><title>CyberSec Lab</title></head>
<body>
  <h1>Welcome to the Security Challenge</h1>
  <p>There's a hidden file somewhere on this site. Find it, download it, and crack it.</p>
  <p>Good luck!</p>
  <!-- hint: always check what a site doesn't want you to see -->
</body>
</html>
```

Push everything to your `username.github.io` repo and enable GitHub Pages.

---

## Part 2: Participant Exercise

### Step 1 — Recon with Gobuster

Scan the target site for hidden directories and files:

```bash
gobuster dir -u https://username.github.io -w /usr/share/wordlists/dirb/common.txt -x zip,txt,html
```

Expected output:

```
/index.html          (Status: 200)
/robots.txt          (Status: 200)
/backup              (Status: 301)
```

### Step 2 — Manual Recon

```bash
# Check robots.txt (always a first step in recon)
curl https://username.github.io/robots.txt

# Output reveals:
# Disallow: /backup/

# View page source for hints
curl https://username.github.io/index.html
```

### Step 3 — Download the ZIP

```bash
wget https://username.github.io/backup/secret.zip
```

### Step 4 — Crack with John the Ripper

```bash
# Extract the hash from the ZIP file
zip2john secret.zip > zip_hash.txt

# Crack using a wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

# View the cracked password
john --show zip_hash.txt

# Unzip with the cracked password
unzip secret.zip
cat flag.txt
# Output: FLAG{y0u_cr4ck3d_1t}
```

---

## Difficulty Levels

Host multiple ZIPs with increasing password strength:

| Level | Password | Wordlist | Crack Time |
|---|---|---|---|
| Easy | `password` | rockyou.txt | < 1 sec |
| Medium | `s3cure123` | rockyou.txt | Few seconds |
| Hard | `Blu3$ky!99` | rockyou.txt | May fail |

---

## Key Teaching Points

- **robots.txt is a roadmap, not a lock** — it tells crawlers what to avoid, but attackers read it first
- **Weak passwords fall to dictionary attacks in seconds** — length and complexity matter
- **ZIP encryption (ZipCrypto) is inherently weak** — modern alternatives like 7z with AES-256 are far stronger
- **Recon is the first phase of any security assessment** — understanding the attack surface before acting

---

## Wordlist Setup

`rockyou.txt` comes preinstalled on Kali Linux. For other OSes:

```bash
wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

---

> **Disclaimer:** These tools and techniques must only be used on systems you own or have explicit written authorization to test. Unauthorized use is illegal under computer fraud laws in most jurisdictions.
