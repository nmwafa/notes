# DNS Information Gathering — Cheatsheet

> Teknik recon, tools, dan commands untuk penetration testing & OSINT  
> Kategori: `passive` `active` `osint` `brute-force` `advanced`
>
> Di Generate oleh Claude Sonnet 4.6 Extended

---

## Daftar Isi

1. [DNS Record Types](#1-dns-record-types)
2. [DNS Zone Transfer (AXFR/IXFR)](#2-dns-zone-transfer-axfrixfr)
3. [Subdomain Enumeration](#3-subdomain-enumeration)
4. [Tools Utama](#4-tools-utama)
5. [OSINT & Passive Recon](#5-osint--passive-recon)
6. [DNS Brute Force](#6-dns-brute-force)
7. [Teknik Advanced](#7-teknik-advanced)
8. [Referensi Wordlist](#8-referensi-wordlist)

---

## 1. DNS Record Types

Query berbagai jenis DNS record menggunakan `dig`, `nslookup`, atau `host`.

### A Record — IPv4 Address

```bash
dig target.com A
nslookup target.com
host target.com
```

### AAAA Record — IPv6 Address

```bash
dig target.com AAAA
```

### MX Record — Mail Servers

```bash
dig target.com MX
nslookup -type=mx target.com
```

### NS Record — Nameservers

```bash
dig target.com NS
nslookup -type=ns target.com
```

### TXT Record — SPF, DKIM, Verification Tokens

```bash
dig target.com TXT
dig _dmarc.target.com TXT
dig _domainkey.target.com TXT
```

### SOA Record — Start of Authority (admin info, serial)

```bash
dig target.com SOA
```

### CNAME Record — Canonical Name / Alias

```bash
dig www.target.com CNAME
```

### PTR Record — Reverse DNS Lookup

```bash
dig -x 1.2.3.4
nslookup 1.2.3.4
host 1.2.3.4
```

### SRV Record — Service Locator (VoIP, XMPP, dll)

```bash
dig _sip._tcp.target.com SRV
dig _xmpp-server._tcp.target.com SRV
```

### ANY — Semua Record

```bash
dig target.com ANY                    # mungkin diblokir di beberapa server
dig target.com ANY +noall +answer
```

### Menggunakan DNS Server Tertentu

```bash
dig target.com @8.8.8.8              # Google DNS
dig target.com @1.1.1.1              # Cloudflare
dig target.com @ns1.target.com       # nameserver target langsung
```

---

## 2. DNS Zone Transfer (AXFR/IXFR)

> **Tingkat Kritis: Tinggi**  
> Jika server DNS misconfigured, zone transfer dapat mendump seluruh zona DNS sekaligus — semua subdomain, IP, MX, TXT, dan struktur jaringan internal target.

### Step 1 — Temukan Nameservers

```bash
dig target.com NS +short
```

### Step 2 — Coba AXFR ke Setiap Nameserver

```bash
dig axfr target.com @ns1.target.com
dig axfr target.com @ns2.target.com
```

### Dengan `host` Command

```bash
host -l target.com ns1.target.com
```

### Dengan `nslookup` (Interactive Mode)

```
nslookup
> server ns1.target.com
> set type=any
> target.com
> ls -d target.com
```

### IXFR — Incremental Zone Transfer

```bash
dig ixfr=0 target.com @ns1.target.com
```

### Automated dengan `fierce`

```bash
fierce --domain target.com
```

### Automated dengan `dnsenum`

```bash
dnsenum --enum target.com
```

**Output yang didapat jika berhasil:** semua A/CNAME/MX/TXT records, internal hostnames, IP range, struktur jaringan internal.

---

## 3. Subdomain Enumeration

### subfinder — Passive, Multi-Source

```bash
subfinder -d target.com
subfinder -d target.com -o subs.txt
subfinder -d target.com -all -recursive
```

### amass — Comprehensive Enumeration

```bash
amass enum -passive -d target.com
amass enum -active -d target.com
amass enum -brute -w wordlist.txt -d target.com
```

### assetfinder — Cepat dan Ringan

```bash
assetfinder --subs-only target.com
assetfinder target.com | grep target.com
```

### findomain — Cepat, Multi-Platform

```bash
findomain -t target.com
findomain -t target.com -u subs.txt
```

### knockpy — Python-Based Enumeration

```bash
knockpy target.com
knockpy target.com -w wordlist.txt
```

### Pipeline — Gabungkan Semua Hasil

```bash
subfinder -d target.com -silent | anew all_subs.txt
assetfinder target.com | anew all_subs.txt
amass enum -passive -d target.com | anew all_subs.txt
```

---

## 4. Tools Utama

### dnsenum — Full Enumeration

```bash
dnsenum target.com
dnsenum --enum -f dns-names.txt -r target.com
dnsenum --noreverse --nocolor target.com -o result.xml
```

### dnsrecon — Comprehensive DNS Recon

```bash
dnsrecon -d target.com
dnsrecon -d target.com -t axfr          # zone transfer
dnsrecon -d target.com -t brt -D list.txt  # brute force
dnsrecon -d target.com -t rvl           # reverse lookup
dnsrecon -d target.com -t std           # standard enumeration
dnsrecon -d target.com -t goo           # google enumeration
dnsrecon -d target.com --xml out.xml
```

Tipe (`-t`) yang tersedia:

| Flag | Deskripsi |
|------|-----------|
| `std` | Standard enumeration (A, AAAA, NS, SOA, MX) |
| `axfr` | Zone transfer ke semua NS |
| `brt` | Brute force subdomains/hosts |
| `rvl` | Reverse lookup pada IP range |
| `goo` | Google enumeration |
| `snoop` | Cache snooping |
| `tld` | TLD expansion |
| `zonewalk` | NSEC zone walking |

### fierce — Domain Recon & Zone Transfer

```bash
fierce --domain target.com
fierce --domain target.com --wordlist hosts.txt
fierce --domain target.com --traverse 10
```

### nmap DNS Scripts

```bash
# Zone transfer
nmap -p 53 --script dns-zone-transfer \
  --script-args dns-zone-transfer.domain=target.com ns1.target.com

# DNS brute force
nmap --script dns-brute target.com

# DNS recursion check
nmap -sU -p 53 --script dns-recursion target.com

# SRV enumeration
nmap --script dns-srv-enum target.com
```

### theHarvester — DNS + Email OSINT

```bash
theHarvester -d target.com -b all
theHarvester -d target.com -b dnsdumpster,google,bing
theHarvester -d target.com -b all -f output.html
```

---

## 5. OSINT & Passive Recon

### Certificate Transparency — crt.sh

Certificate Transparency log menyimpan semua SSL/TLS cert yang pernah diissue. Sangat efektif untuk menemukan subdomain tersembunyi.

```bash
# Via browser
https://crt.sh/?q=%.target.com

# Via curl + jq
curl -s "https://crt.sh/?q=%.target.com&output=json" \
  | jq -r '.[].name_value' | sort -u

# Tanpa jq
curl -s "https://crt.sh/?q=target.com&output=json" \
  | python3 -c "import sys,json; [print(i['name_value']) for i in json.load(sys.stdin)]"
```

### Passive DNS Databases

| Platform | URL / Cara Akses |
|----------|------------------|
| **DNSDumpster** | `dnsdumpster.com` — visual DNS map + export CSV |
| **SecurityTrails** | `securitytrails.com/domain/target.com/dns` |
| **VirusTotal** | `virustotal.com/gui/domain/target.com/relations` |
| **Shodan** | `shodan.io/search?query=hostname:target.com` |
| **Censys** | `search.censys.io` — DNS + TLS fingerprint |
| **RiskIQ / Silentpush** | Passive DNS, WHOIS, historical data |
| **DNSDB (Farsight)** | Passive DNS database terbesar |
| **HackerTarget** | `hackertarget.com/find-dns-host-records/` |

### VirusTotal API

```bash
curl -H "x-apikey: YOUR_API_KEY" \
  "https://www.virustotal.com/api/v3/domains/target.com/subdomains"
```

### WHOIS & RDAP

```bash
whois target.com
whois 1.2.3.4                                    # reverse WHOIS
curl "https://rdap.org/domain/target.com"
```

Tools tambahan: `jwhois`, `python-whois`, domaintools.com

### Wayback Machine / Archive

```bash
# CDX API — temukan subdomain dari URL yang pernah di-crawl
curl "http://web.archive.org/cdx/search/cdx?\
url=*.target.com/*&output=text&fl=original&collapse=urlkey"

# gau — Get All URLs
gau target.com | unfurl --unique domains
```

---

## 6. DNS Brute Force

### gobuster dns — Brute Force Subdomain

```bash
gobuster dns -d target.com -w /usr/share/wordlists/dns.txt
gobuster dns -d target.com -w list.txt -t 50 -r 8.8.8.8
```

### ffuf — DNS Fuzzing

```bash
# Subdomain fuzzing
ffuf -w subs.txt -u https://FUZZ.target.com -mc 200,301,302

# Virtual host fuzzing
ffuf -w subs.txt -H "Host: FUZZ.target.com" -u http://IP_TARGET
```

### massdns — Ultra Fast (Jutaan Query/Detik)

```bash
# Buat daftar targets
cat wordlist.txt | sed 's/$/.target.com/' > targets.txt

# Jalankan massdns
massdns -r resolvers.txt -t A -o S targets.txt > output.txt
```

### puredns — Resolusi Massal + Brute Force

```bash
puredns bruteforce wordlist.txt target.com
puredns resolve subs.txt -r resolvers.txt
```

### shuffledns — Wrapper massdns + Validasi

```bash
shuffledns -d target.com -w subs.txt -r resolvers.txt
```

### dnsmap — Brute Force + Reverse Lookup

```bash
dnsmap target.com
dnsmap target.com -w wordlist.txt
```

---

## 7. Teknik Advanced

### DNS Cache Snooping

Mengetahui apakah suatu domain pernah diakses oleh pengguna server DNS target (non-recursive query).

```bash
dig +norecurse target.com @DNS_SERVER
# Jika TTL tidak = 0 → ada di cache → pernah diakses
```

### Reverse DNS Sweep (PTR)

```bash
# Bash loop manual
for i in {1..254}; do host 192.168.1.$i; done

# dnsrecon
dnsrecon -r 192.168.1.0/24 -n 8.8.8.8

# nmap list scan + rDNS
nmap -sL 192.168.1.0/24
```

### Subdomain Takeover Detection

Subdomain Takeover terjadi saat CNAME mengarah ke layanan cloud yang tidak aktif (Heroku, GitHub Pages, S3, Azure, dll).

```bash
# subjack
subjack -w subs.txt -t 100 -timeout 30 -ssl -c fingerprints.json

# nuclei (template takeovers)
nuclei -l subs.txt -t ~/nuclei-templates/takeovers/

# subzy
subzy run --targets subs.txt
```

### DNSSEC Enumeration — NSEC Zone Walking

NSEC records di DNSSEC dapat digunakan untuk meng-enumerate semua record dalam zona.

```bash
dig target.com NSEC
ldns-walk @ns1.target.com target.com    # NSEC zone walk
nsec3walker target.com                   # NSEC3 hash crack
```

### DNS over HTTPS (DoH) — Query via API

```bash
# Cloudflare DoH
curl -H 'accept: application/dns-json' \
  'https://1.1.1.1/dns-query?name=target.com&type=A'

# Google DoH
curl 'https://dns.google/resolve?name=target.com&type=ANY'
```

### ASN & IP Range Discovery

```bash
whois -h whois.radb.net target.com
curl "https://api.bgpview.io/search?query_term=target.com"

# Amass intel
amass intel -org "Target Corp"    # ASN dari nama organisasi
amass intel -asn 12345            # IP range dari ASN
amass intel -d target.com -whois  # WHOIS-based discovery
```

### DNS Amplification Check (Recursion Test)

```bash
dig +short test.openresolver.com TXT @DNS_SERVER
nmap -sU -p 53 --script dns-recursion DNS_SERVER
```

### IPv6 DNS Recon

```bash
dig target.com AAAA
dig -6 target.com @2001:4860:4860::8888    # via IPv6 Google DNS
```

---

## 8. Referensi Wordlist

### SecLists (Rekomendasi Utama)

```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
/usr/share/seclists/Discovery/DNS/dns-Jhaddix.txt
/usr/share/seclists/Discovery/DNS/fierce-hostlist.txt
```

### Install SecLists

```bash
sudo apt install seclists
# atau
git clone https://github.com/danielmiessler/SecLists
```

### Resolvers List untuk massdns / puredns

```bash
# Download resolvers valid
curl -s https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt -o resolvers.txt
```

---

## Quick Reference — Urutan Eksekusi Recon

```
1. WHOIS & NS discovery       →  whois, dig NS
2. Zone Transfer              →  dig axfr, dnsenum
3. Passive subdomain enum     →  subfinder, assetfinder, crt.sh
4. Active DNS brute force     →  puredns, massdns, gobuster dns
5. Resolve & validasi         →  puredns resolve, httpx
6. Reverse DNS sweep          →  dnsrecon -r, nmap -sL
7. Takeover check             →  nuclei, subjack
8. Advanced (NSEC, cache)     →  ldns-walk, dig +norecurse
```
