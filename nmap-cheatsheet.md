# Nmap Cheatsheet
> Network Mapper — Panduan Lengkap dari Basic sampai Advanced
>
> Di Generate oleh Claude Sonnet 4.6 Extended

---

## Daftar Isi
- [Host Discovery](#host-discovery)
- [Port Scan](#port-scan)
- [Tipe Scan](#tipe-scan)
- [Service & Version Detection](#service--version-detection)
- [OS Detection](#os-detection)
- [NSE — Nmap Scripting Engine](#nse--nmap-scripting-engine)
- [NSE — Script Kategori Populer](#nse--script-kategori-populer)
- [Timing & Performa](#timing--performa)
- [Format Output](#format-output)
- [Firewall/IDS Evasion](#firewallids-evasion)
- [Teknik Lanjutan](#teknik-lanjutan)

---

## Host Discovery
> Level: **Basic**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap 192.168.1.1` | Scan satu host/IP |
| `nmap 192.168.1.1-254` | Scan range IP |
| `nmap 192.168.1.0/24` | Scan seluruh subnet /24 |
| `nmap -sn 192.168.1.0/24` | Ping scan — hanya cek host aktif, tanpa port scan |
| `nmap -Pn 192.168.1.1` | Skip host discovery, anggap semua host hidup |
| `nmap -PS 192.168.1.1` | TCP SYN ping discovery |
| `nmap -PA 192.168.1.1` | TCP ACK ping discovery |
| `nmap -PU 192.168.1.1` | UDP ping discovery |
| `nmap -PE 192.168.1.1` | ICMP echo request ping |
| `nmap --traceroute 192.168.1.1` | Trace rute ke host target |
| `nmap -iL targets.txt` | Baca daftar target dari file |
| `nmap --exclude 192.168.1.5 192.168.1.0/24` | Kecualikan IP tertentu dari scan |

---

## Port Scan
> Level: **Basic**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap -p 22 192.168.1.1` | Scan port spesifik |
| `nmap -p 22,80,443 192.168.1.1` | Scan beberapa port sekaligus |
| `nmap -p 1-1000 192.168.1.1` | Scan range port |
| `nmap -p- 192.168.1.1` | Scan semua 65535 port |
| `nmap --top-ports 100 192.168.1.1` | Scan 100 port paling umum |
| `nmap -F 192.168.1.1` | Fast scan — 100 port top (lebih cepat) |
| `nmap -p U:53,T:80 192.168.1.1` | Scan UDP port 53 dan TCP port 80 |

---

## Tipe Scan
> Level: **Intermediate**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap -sS 192.168.1.1` | TCP SYN scan (stealth) — default jika root |
| `nmap -sT 192.168.1.1` | TCP connect scan — untuk non-root user |
| `nmap -sU 192.168.1.1` | UDP scan — lebih lambat, perlu root |
| `nmap -sA 192.168.1.1` | TCP ACK scan — deteksi firewall rules |
| `nmap -sW 192.168.1.1` | TCP Window scan |
| `nmap -sN 192.168.1.1` | TCP Null scan — tidak ada flag TCP |
| `nmap -sF 192.168.1.1` | TCP FIN scan — hanya flag FIN |
| `nmap -sX 192.168.1.1` | Xmas scan — FIN, URG, PSH flags aktif |
| `nmap -sM 192.168.1.1` | TCP Maimon scan |
| `nmap -sI zombie 192.168.1.1` | Idle/Zombie scan — IP spoofing via zombie host |

---

## Service & Version Detection
> Level: **Intermediate**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap -sV 192.168.1.1` | Deteksi versi service yang berjalan |
| `nmap -sV --version-intensity 9 192.168.1.1` | Version detection intensitas maksimal (0-9) |
| `nmap -sV --version-light 192.168.1.1` | Version detection ringan (intensitas 2) |
| `nmap -sV --version-all 192.168.1.1` | Coba semua probe untuk version detection |
| `nmap -A 192.168.1.1` | Aggressive: OS, version, script, traceroute sekaligus |

---

## OS Detection
> Level: **Intermediate**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap -O 192.168.1.1` | OS detection (perlu root/admin) |
| `nmap -O --osscan-guess 192.168.1.1` | Tebak OS walau tidak pasti |
| `nmap -O --osscan-limit 192.168.1.1` | Hanya OS scan jika ada port terbuka & tertutup |
| `nmap -O --max-os-tries 3 192.168.1.1` | Batasi percobaan OS detection |

---

## NSE — Nmap Scripting Engine
> Level: **Intermediate**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap -sC 192.168.1.1` | Jalankan default scripts (sama dengan `--script=default`) |
| `nmap --script=http-title 192.168.1.1` | Jalankan script spesifik |
| `nmap --script=http-* 192.168.1.1` | Jalankan semua script HTTP |
| `nmap --script=vuln 192.168.1.1` | Cek kerentanan umum |
| `nmap --script=auth 192.168.1.1` | Script autentikasi (brute force, bypass) |
| `nmap --script=safe 192.168.1.1` | Hanya jalankan script yang aman |
| `nmap --script=banner 192.168.1.1` | Ambil banner dari service |
| `nmap --script=smb-vuln-* 192.168.1.1` | Cek kerentanan SMB (EternalBlue dll) |
| `nmap --script=ssl-enum-ciphers -p 443 192.168.1.1` | Enum cipher suite SSL/TLS |
| `nmap --script-args='user=admin,pass=1234' --script=http-auth 192.168.1.1` | Kirim argumen ke script |
| `nmap --script-help http-title` | Lihat dokumentasi script tertentu |
| `nmap --script-updatedb` | Update database script NSE |

---

## NSE — Script Kategori Populer
> Level: **Advanced**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap --script=discovery 192.168.1.1` | Kumpulkan informasi tambahan tentang host |
| `nmap --script=exploit 192.168.1.1` | Script eksploitasi (gunakan dengan izin!) |
| `nmap --script=brute 192.168.1.1` | Brute force login berbagai protokol |
| `nmap --script=dos 192.168.1.1` | Script DoS — sangat hati-hati! |
| `nmap --script=intrusive 192.168.1.1` | Script invasif — bisa crash service target |
| `nmap --script=ftp-anon -p 21 192.168.1.1` | Cek akses FTP anonim |
| `nmap --script=dns-brute --script-args dns-brute.domain=target.com` | DNS subdomain brute force |

---

## Timing & Performa
> Level: **Intermediate**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap -T0 192.168.1.1` | Paranoid — sangat lambat, IDS evasion |
| `nmap -T1 192.168.1.1` | Sneaky — lambat untuk evasion |
| `nmap -T2 192.168.1.1` | Polite — hemat bandwidth |
| `nmap -T3 192.168.1.1` | Normal — default |
| `nmap -T4 192.168.1.1` | Aggressive — lebih cepat, jaringan cepat |
| `nmap -T5 192.168.1.1` | Insane — sangat cepat, mungkin miss port |
| `nmap --min-rate 1000 192.168.1.1` | Kirim minimal 1000 paket per detik |
| `nmap --max-retries 1 192.168.1.1` | Batasi retry per port |
| `nmap --host-timeout 30s 192.168.1.1` | Timeout per host 30 detik |

---

## Format Output
> Level: **Basic**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap -oN output.txt 192.168.1.1` | Simpan output normal ke file |
| `nmap -oX output.xml 192.168.1.1` | Simpan output format XML |
| `nmap -oG output.gnmap 192.168.1.1` | Simpan output grepable |
| `nmap -oA output 192.168.1.1` | Simpan semua format sekaligus (.nmap, .xml, .gnmap) |
| `nmap -v 192.168.1.1` | Verbose — tampilkan lebih banyak info |
| `nmap -vv 192.168.1.1` | Very verbose |
| `nmap -d 192.168.1.1` | Debug mode |
| `nmap --open 192.168.1.1` | Tampilkan hanya port yang terbuka |
| `nmap --reason 192.168.1.1` | Tampilkan alasan status port |
| `nmap --packet-trace 192.168.1.1` | Tampilkan semua paket yang dikirim/diterima |

---

## Firewall/IDS Evasion
> Level: **Advanced**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap -f 192.168.1.1` | Fragment paket — bypass beberapa firewall |
| `nmap --mtu 24 192.168.1.1` | Set ukuran MTU custom untuk fragmentasi |
| `nmap -D RND:10 192.168.1.1` | Decoy: 10 IP palsu acak + IP asli |
| `nmap -D 10.0.0.1,10.0.0.2 192.168.1.1` | Decoy: IP palsu spesifik |
| `nmap -S 10.0.0.99 192.168.1.1` | Spoof source IP address |
| `nmap -e eth0 192.168.1.1` | Gunakan interface jaringan tertentu |
| `nmap --source-port 53 192.168.1.1` | Spoof source port (misal port DNS 53) |
| `nmap --data-length 25 192.168.1.1` | Tambah data acak di paket |
| `nmap --badsum 192.168.1.1` | Kirim paket dengan checksum rusak |
| `nmap --proxies http://proxy:8080 192.168.1.1` | Gunakan proxy/rantai proxy |
| `nmap --spoof-mac 0 192.168.1.1` | Spoof MAC address acak |
| `nmap --spoof-mac Dell 192.168.1.1` | Spoof MAC sesuai vendor |

---

## Teknik Lanjutan
> Level: **Advanced**

| Perintah | Deskripsi |
|----------|-----------|
| `nmap --script=http-enum -p 80,443 192.168.1.1` | Enum direktori & file web umum |
| `nmap -sV -sC -p- --min-rate 5000 192.168.1.1` | Full scan cepat untuk pentest (semua port) |
| `nmap -sn --script=nbstat 192.168.1.0/24` | Ambil nama NetBIOS seluruh jaringan |
| `nmap -6 fe80::1` | Scan target IPv6 |
| `nmap --script=broadcast-dhcp-discover` | Discover DHCP server di jaringan lokal |
| `nmap --script=targets-ipv6-multicast-echo` | Discover host IPv6 via multicast |
| `nmap --script=firewalk --traceroute 192.168.1.1` | Identifikasi port yang di-filter firewall |
| `nmap --scanflags URGACKPSHRSTSYNFIN 192.168.1.1` | Custom TCP flags untuk scan |
| `nmap --ip-options 'L 192.168.1.1 192.168.1.2' 192.168.1.1` | Gunakan IP options untuk loose source routing |
