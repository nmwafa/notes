---
title: "Nmap Cheatsheet"
layout: default
---

# Cheatsheet Nmap Lengkap

## Daftar Isi

1. [Pengenalan Nmap](#1-pengenalan-nmap)
2. [Instalasi](#2-instalasi)
3. [Sintaks Dasar](#3-sintaks-dasar)
4. [Spesifikasi Target](#4-spesifikasi-target)
5. [Host Discovery](#5-host-discovery)
6. [Status dan State Port](#6-status-dan-state-port)
7. [Teknik Scanning](#7-teknik-scanning)
8. [Spesifikasi Port](#8-spesifikasi-port)
9. [Deteksi Service dan Versi](#9-deteksi-service-dan-versi)
10. [Deteksi Sistem Operasi (OS)](#10-deteksi-sistem-operasi-os)
11. [Timing dan Performance](#11-timing-dan-performance)
12. [Nmap Scripting Engine (NSE)](#12-nmap-scripting-engine-nse)
13. [Firewall dan IDS Evasion](#13-firewall-dan-ids-evasion)
14. [Format Output](#14-format-output)
15. [Kombinasi Options Populer](#15-kombinasi-options-populer)
16. [Skenario Dunia Nyata](#16-skenario-dunia-nyata)
17. [Tabel Referensi Cepat](#17-tabel-referensi-cepat)

---

## 1. Pengenalan Nmap

Nmap digunakan untuk:

- Menemukan host yang aktif di sebuah jaringan
- Mendeteksi port yang terbuka pada host tersebut
- Mengidentifikasi service dan versi software yang berjalan
- Menebak sistem operasi target
- Menjalankan script otomatis (NSE) untuk audit konfigurasi dan vulnerability

**Tools pendukung satu paket dengan nmap:**

| Tool | Fungsi |
|---|---|
| `ncat` | Pengganti modern `netcat`, untuk koneksi TCP/UDP manual |
| `nping` | Pembuat paket kustom untuk uji ping/traceroute |
| `ndiff` | Membandingkan dua hasil scan nmap |
| `zenmap` | GUI resmi untuk nmap |

---

## 2. Instalasi

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install nmap -y

# CentOS / RHEL / Fedora
sudo dnf install nmap -y

# macOS (Homebrew)
brew install nmap

# Verifikasi instalasi
nmap --version
```

Windows: unduh installer dari situs resmi nmap.org.

---

## 3. Sintaks Dasar

```bash
nmap [Jenis Scan] [Options] {target}
```

Contoh paling sederhana:

```bash
nmap 192.168.1.1
nmap scanme.nmap.org
```

Jika jenis scan tidak disebutkan, nmap secara default menggunakan:
- `-sS` (SYN scan) bila dijalankan dengan `sudo`/root
- `-sT` (TCP connect scan) bila dijalankan tanpa privilege root
- 1000 port yang paling umum digunakan

**Catatan tambahan:**
- `-6` → aktifkan scanning IPv6
- Sebagian besar teknik scan (`-sS`, `-sU`, `-O`, dll) membutuhkan privilege root/administrator

---

## 4. Spesifikasi Target

| Perintah | Keterangan |
|---|---|
| `nmap 192.168.1.1` | Scan satu alamat IP |
| `nmap example.com` | Scan berdasarkan hostname/domain |
| `nmap 192.168.1.1 192.168.1.5` | Scan beberapa target sekaligus |
| `nmap 192.168.1.1-50` | Scan range IP |
| `nmap 192.168.1.0/24` | Scan seluruh subnet (notasi CIDR) |
| `nmap -iL targets.txt` | Ambil daftar target dari file |
| `nmap -iR 100` | Scan 100 target acak di internet |
| `nmap 192.168.1.0/24 --exclude 192.168.1.5` | Scan subnet tapi kecualikan IP tertentu |
| `nmap -iL targets.txt --excludefile skip.txt` | Kecualikan target dari file |

Contoh isi `targets.txt`:
```
192.168.1.1
192.168.1.10-20
10.0.0.0/24
scanme.nmap.org
```

---

## 5. Host Discovery

Tahap awal untuk menentukan host mana yang aktif, sebelum scan port dijalankan.

| Option | Fungsi |
|---|---|
| `-sn` | Ping scan saja, **tanpa** scan port |
| `-Pn` | Lewati host discovery, anggap semua target aktif (berguna jika ICMP diblokir) |
| `-sL` | List scan — hanya menampilkan daftar target, tidak mengirim paket sama sekali |
| `-PE` | ICMP Echo request (ping standar) |
| `-PP` | ICMP Timestamp request |
| `-PM` | ICMP Address Mask request |
| `-PS<port>` | TCP SYN discovery ke port tertentu (default 80) |
| `-PA<port>` | TCP ACK discovery |
| `-PU<port>` | UDP discovery |
| `-PR` | ARP discovery — otomatis dipakai & paling akurat di jaringan lokal |
| `-n` | Nonaktifkan DNS resolution (lebih cepat) |
| `-R` | Paksa DNS resolution untuk semua target |

```bash
# Cari host aktif di jaringan lokal tanpa scan port
nmap -sn 192.168.1.0/24

# Paksa scan meski target tidak merespon ping
nmap -Pn 192.168.1.10
```

---

## 6. Status dan State Port

Setiap port yang di-scan akan diberi salah satu status berikut:

| State | Arti |
|---|---|
| `open` | Ada aplikasi yang aktif menerima koneksi di port ini |
| `closed` | Port terjangkau, tapi tidak ada aplikasi yang listen |
| `filtered` | Firewall/filter memblokir probe, nmap tidak bisa memastikan status |
| `unfiltered` | Port terjangkau tapi nmap tidak bisa pastikan open/closed (muncul di ACK scan) |
| `open\|filtered` | Nmap tidak bisa membedakan open atau filtered (umum di scan UDP/NULL/FIN/Xmas) |
| `closed\|filtered` | Nmap tidak bisa membedakan closed atau filtered (muncul di idle scan) |

---

## 7. Teknik Scanning

| Option | Nama | Keterangan |
|---|---|---|
| `-sS` | SYN Scan | "Stealth scan", paling umum & cepat, tidak menyelesaikan TCP handshake. Butuh root |
| `-sT` | TCP Connect Scan | Menyelesaikan full 3-way handshake, tidak butuh root, tapi lebih mudah tercatat di log |
| `-sU` | UDP Scan | Scan port UDP; jauh lebih lambat karena sifat UDP yang connectionless |
| `-sA` | ACK Scan | Untuk memetakan aturan firewall, bukan mencari port terbuka |
| `-sW` | Window Scan | Mirip ACK scan, memanfaatkan TCP window size untuk membedakan state |
| `-sM` | Maimon Scan | Kombinasi flag FIN/ACK, memanfaatkan bug lama pada BSD |
| `-sN` | Null Scan | Paket tanpa flag TCP sama sekali, kadang bisa lolos firewall sederhana |
| `-sF` | FIN Scan | Paket dengan flag FIN saja |
| `-sX` | Xmas Scan | Paket dengan flag FIN + PSH + URG menyala bersamaan |
| `-sO` | IP Protocol Scan | Deteksi protokol IP yang didukung host (TCP, UDP, ICMP, GRE, dll) |
| `-sY` | SCTP INIT Scan | Untuk protokol SCTP |
| `-sZ` | SCTP COOKIE ECHO Scan | Varian stealth untuk SCTP |
| `-sI <zombie>` | Idle/Zombie Scan | Scan via host pihak ketiga agar identitas asli tersembunyi |

```bash
# SYN scan standar
sudo nmap -sS 192.168.1.10

# TCP connect scan tanpa root
nmap -sT 192.168.1.10

# UDP scan pada port yang paling umum
sudo nmap -sU --top-ports 20 192.168.1.10

# Scan TCP + UDP dalam satu perintah
sudo nmap -sS -sU -p T:22,80,443,U:53,161 192.168.1.10
```

---

## 8. Spesifikasi Port

| Option | Fungsi |
|---|---|
| `-p 80` | Scan hanya port 80 |
| `-p 80,443,8080` | Scan beberapa port spesifik |
| `-p 1-1000` | Scan range port |
| `-p-` | Scan semua 65535 port |
| `-p U:53,111,T:21-25,80` | Kombinasi TCP dan UDP (perlu `-sS -sU` bersamaan) |
| `-F` | Fast scan — 100 port tersering |
| `--top-ports 50` | Scan N port paling sering dipakai |
| `-r` | Scan port berurutan, tidak diacak |

```bash
nmap -p 80,443,8080,8443 192.168.1.10
nmap -p- -T4 192.168.1.10
nmap --top-ports 50 192.168.1.10
```

---

## 9. Deteksi Service dan Versi

| Option | Fungsi |
|---|---|
| `-sV` | Deteksi versi service pada port yang terbuka |
| `--version-intensity <0-9>` | Level intensitas probe (0 = ringan, 9 = lengkap) |
| `--version-light` | Setara intensity 2, cepat |
| `--version-all` | Setara intensity 9, paling akurat tapi paling lambat |
| `-A` | Aggressive scan: gabungan `-O -sV -sC --traceroute` |

```bash
nmap -sV 192.168.1.10
nmap -sV --version-intensity 9 192.168.1.10
sudo nmap -A 192.168.1.10
```

---

## 10. Deteksi Sistem Operasi (OS)

| Option | Fungsi |
|---|---|
| `-O` | Aktifkan deteksi OS (idealnya ada 1 port open & 1 closed untuk akurasi terbaik) |
| `--osscan-limit` | Hanya coba deteksi OS pada target dengan kandidat kuat |
| `--osscan-guess` | Tebak OS lebih agresif walau data tidak sempurna |
| `--max-os-tries <n>` | Batasi jumlah percobaan deteksi OS |

```bash
sudo nmap -O 192.168.1.10
sudo nmap -O --osscan-guess 192.168.1.10
```

---

## 11. Timing dan Performance

Nmap punya 6 template kecepatan, dari paling hati-hati sampai paling agresif:

| Template | Nama | Kapan dipakai |
|---|---|---|
| `-T0` | Paranoid | Sangat lambat, evasion IDS maksimal (bisa berjam-jam) |
| `-T1` | Sneaky | Lambat, untuk evasion |
| `-T2` | Polite | Lebih pelan dari default, hemat bandwidth |
| `-T3` | Normal | **Default** nmap |
| `-T4` | Aggressive | Umum dipakai untuk jaringan stabil/cepat |
| `-T5` | Insane | Tercepat, berisiko banyak hasil tidak akurat |

Fine-tuning manual:
```bash
--min-rate 300        # minimal paket per detik
--max-rate 1000       # maksimal paket per detik
--min-parallelism 10  # jumlah probe paralel minimum
--max-retries 2       # maksimal retransmisi paket
--host-timeout 30m    # skip host jika kelamaan
--scan-delay 1s        # jeda antar probe (evasion)
```

```bash
# Scan cepat untuk jaringan lokal
nmap -T4 -F 192.168.1.0/24

# Scan pelan untuk menghindari deteksi IDS
nmap -T2 --scan-delay 2s 192.168.1.10
```

---

## 12. Nmap Scripting Engine (NSE)

NSE menjalankan script Lua bawaan nmap untuk otomatisasi discovery, audit, hingga deteksi vulnerability.

| Option | Fungsi |
|---|---|
| `-sC` | Jalankan script kategori `default` |
| `--script=<nama>` | Jalankan script spesifik |
| `--script=<kategori>` | Jalankan semua script dalam satu kategori |
| `--script-args=<args>` | Kirim argumen ke script |
| `--script-help=<nama>` | Lihat dokumentasi sebuah script |
| `--script-updatedb` | Update database script |

**Kategori NSE:**

| Kategori | Keterangan |
|---|---|
| `auth` | Cek mekanisme autentikasi |
| `broadcast` | Discovery via broadcast di LAN |
| `brute` | Brute-force credential |
| `default` | Script standar (dipakai oleh `-sC`) |
| `discovery` | Menggali info tambahan jaringan/service |
| `dos` | Uji denial-of-service (berisiko membuat service crash) |
| `exploit` | Eksploitasi vulnerability yang sudah diketahui |
| `external` | Melibatkan database/resource eksternal |
| `fuzzer` | Fuzzing terhadap service |
| `intrusive` | Berpotensi mengganggu target |
| `malware` | Deteksi indikasi backdoor/malware |
| `safe` | Aman, tidak mengganggu target |
| `version` | Membantu deteksi versi (dipakai oleh `-sV`) |
| `vuln` | Cek vulnerability spesifik yang diketahui |

```bash
# Script default
nmap -sC 192.168.1.10

# Cek vulnerability
nmap -sV --script vuln 192.168.1.10

# Script tertentu
nmap --script http-title 192.168.1.10

# Semua script berawalan "http-"
nmap --script "http-*" 192.168.1.10

# Script dengan argumen (brute-force login)
nmap --script http-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.1.10

# Cek kerentanan SMB terkenal (EternalBlue)
nmap -p445 --script smb-vuln-ms17-010 192.168.1.10
```

---

## 13. Firewall dan IDS Evasion

> Teknik berikut ditujukan untuk pengujian keamanan yang sah (authorized pentest), bukan untuk menyusup ke sistem tanpa izin.

| Option | Fungsi |
|---|---|
| `-f` | Fragmentasi paket agar lolos filter sederhana |
| `--mtu <n>` | Set ukuran MTU kustom (kelipatan 8) |
| `-D decoy1,decoy2,ME` | Scan dengan decoy IP palsu agar sumber tersamar |
| `-D RND:5` | Buat 5 decoy acak otomatis |
| `-S <IP>` | Spoof source IP (butuh kontrol routing) |
| `-e <interface>` | Tentukan network interface yang dipakai |
| `-g <port>` / `--source-port <port>` | Spoof source port (mis. 53, sering dipercaya firewall) |
| `--data-length <n>` | Tambahkan data random ke paket |
| `--spoof-mac <MAC/vendor/0>` | Spoof MAC address (0 = acak penuh) |
| `--badsum` | Kirim checksum salah untuk uji reaksi firewall/IDS |

```bash
sudo nmap -f 192.168.1.10
sudo nmap -D 10.0.0.1,10.0.0.2,ME 192.168.1.10
sudo nmap -g 53 192.168.1.10
sudo nmap --spoof-mac Apple 192.168.1.10
```

---

## 14. Format Output

| Option | Fungsi |
|---|---|
| `-oN file.txt` | Simpan output normal (seperti tampilan terminal) |
| `-oX file.xml` | Simpan output XML (untuk diparsing tools lain) |
| `-oG file.gnmap` | Simpan output grepable |
| `-oA basename` | Simpan ke 3 format sekaligus (.nmap/.xml/.gnmap) |
| `-v` / `-vv` | Verbose / very verbose |
| `-d` / `-dd` | Mode debug |
| `--reason` | Tampilkan alasan status port (mis. "syn-ack") |
| `--open` | Hanya tampilkan port yang open |
| `--stats-every 10s` | Tampilkan progress tiap interval waktu |
| `--traceroute` | Sertakan hasil traceroute ke target |
| `--packet-trace` | Tampilkan tiap paket yang dikirim/diterima (debug mendalam) |

```bash
nmap -A -oA hasil_scan 192.168.1.10
nmap --open 192.168.1.0/24
nmap -v --stats-every 10s -p- 192.168.1.10
```

---

## 15. Kombinasi Options Populer

```bash
# 1. Quick scan — cek cepat port umum
nmap -T4 -F 192.168.1.10

# 2. Scan standar pentest: versi + OS + script default
sudo nmap -sS -sV -O -sC -T4 192.168.1.10

# 3. Full aggressive scan, semua port
sudo nmap -A -T4 -p- 192.168.1.10

# 4. Stealth scan, evasion dasar
sudo nmap -sS -T2 -f --data-length 20 192.168.1.10

# 5. Vulnerability assessment + simpan laporan
nmap -sV --script vuln -oN vuln_report.txt 192.168.1.10

# 6. Discovery jaringan lokal + resolusi hostname
nmap -sn -R 192.168.1.0/24

# 7. Scan semua port + service + script, output lengkap
sudo nmap -p- -sV -sC -oA full_scan 192.168.1.10

# 8. Scan UDP top port (sering terlewat pentester pemula)
sudo nmap -sU --top-ports 100 -sV 192.168.1.10

# 9. Pemetaan aturan firewall
sudo nmap -sA -p 1-1000 192.168.1.10

# 10. Scan subnet besar dengan rate tinggi (jaringan sendiri)
sudo nmap -sS -T4 --min-rate 1000 -p- 192.168.1.0/24
```

---

## 16. Skenario Dunia Nyata

**A. Menemukan semua device aktif di jaringan rumah/kantor**
```bash
sudo nmap -sn 192.168.1.0/24
```

**B. Audit keamanan dasar server web milik sendiri**
```bash
sudo nmap -sV -sC -p 80,443 --script "http-vuln*" example.com
```

**C. Cek apakah server rentan EternalBlue (MS17-010)**
```bash
sudo nmap -p445 --script smb-vuln-ms17-010 192.168.1.10
```

**D. Inventaris service seluruh subnet untuk dokumentasi jaringan**
```bash
sudo nmap -sV -O -oA inventaris_jaringan 192.168.1.0/24
```

**E. Simulasi pentest menyeluruh (dengan izin resmi/scope tertulis)**
```bash
sudo nmap -sS -sV -O -A --script vuln -oA laporan_pentest 192.168.1.10
```

**F. Latihan tanpa risiko hukum**
```bash
# scanme.nmap.org disediakan resmi oleh tim nmap untuk latihan publik
nmap -A scanme.nmap.org
```

---

## 17. Tabel Referensi Cepat

| Kategori | Option Kunci | Contoh Singkat |
|---|---|---|
| Scan dasar | `-sS` `-sT` `-sU` | `nmap -sS target` |
| Target | `-iL` `--exclude` | `nmap -iL list.txt` |
| Discovery | `-sn` `-Pn` | `nmap -sn 10.0.0.0/24` |
| Port | `-p` `-p-` `-F` `--top-ports` | `nmap -p 1-100 target` |
| Deteksi | `-sV` `-O` `-A` | `nmap -A target` |
| Script | `-sC` `--script` | `nmap --script vuln target` |
| Timing | `-T0` … `-T5` | `nmap -T4 target` |
| Evasion | `-f` `-D` `-g` `--spoof-mac` | `nmap -f -D RND:5 target` |
| Output | `-oN` `-oX` `-oA` `--open` | `nmap -oA hasil target` |
