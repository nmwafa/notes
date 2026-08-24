---
title: "Linux Process Termination Cheatsheet"
layout: default
---

# Linux Process Termination Cheatsheet

## 1. Konsep Dasar

Perintah `kill` di Linux sebenarnya **bukan hanya untuk mematikan proses**, melainkan untuk **mengirimkan sinyal (signals)** ke suatu proses. 

* **PID (Process ID):** Angka unik pengidentifikasi proses yang sedang berjalan.
* **Sinyal Default:** Jika tidak menyertakan jenis sinyal, perintah `kill` secara default akan mengirimkan sinyal `SIGTERM` (15), yang meminta proses untuk berhenti secara normal (*graceful shutdown*).

---

## 2. Daftar Sinyal Utama (Signals)

Melihat daftar seluruh sinyal yang didukung sistem:
```bash
kill -l
```

| Nomor | Nama Sinyal | Deskripsi Singkat | Dapat Ditangkap/Dicegah? |
| :---: | :--- | :--- | :---: |
| **1** | `SIGHUP` | Menghubungkan ulang/reload konfigurasi daemon tanpa restart. | Ya |
| **2** | `SIGINT` | Interrupt dari keyboard (sama seperti menekan `Ctrl + C`). | Ya |
| **3** | `SIGQUIT` | Berhenti dan menghasilkan core dump (sama seperti `Ctrl + \`). | Ya |
| **9** | `SIGKILL` | Mematikan proses **seketika secara paksa**. | **TIDAK** |
| **15** | `SIGTERM` | *Default*. Meminta proses berhenti dengan aman (membersihkan cache/file). | Ya |
| **18** | `SIGCONT` | Melanjutkan proses yang sedang di-*pause* / ditangguhkan. | Ya |
| **19** | `SIGSTOP` | Menangguhkan / menjeda (*pause*) proses seketika. | **TIDAK** |
| **20** | `SIGTSTP` | Suspend dari terminal (sama seperti menekan `Ctrl + Z`). | Ya |

---

## 3. Perintah `kill` (Berdasarkan PID)

### A. Menemukan PID Proses
Sebelum menjalankan `kill`, temukan PID terlebih dahulu:
```bash
# Menggunakan pgrep
pgrep -l nginx
pgrep -fa python

# Menggunakan ps & grep
ps aux | grep nginx

# Menggunakan pidof
pidof nginx
```

### B. Sintaks & Contoh Penggunaan

```bash
# 1. Menghentikan proses secara normal (SIGTERM / 15)
kill <PID>
kill -15 <PID>
kill -TERM <PID>
kill -s SIGTERM <PID>

# 2. Menghentikan proses secara paksa (SIGKILL / 9)
kill -9 <PID>
kill -KILL <PID>
kill -s SIGKILL <PID>

# 3. Reload konfigurasi layanan (SIGHUP / 1)
kill -1 <PID>
kill -HUP <PID>

# 4. Menjeda dan Melanjutkan Proses
kill -STOP <PID>   # Menjeda proses (pause)
kill -CONT <PID>   # Melanjutkan proses (resume)

# 5. Membunuh beberapa PID sekaligus
kill 1234 5678 9101
```

---

## 4. Perintah `pkill` (Berdasarkan Pola / Nama)

`pkill` memungkinkan Anda mengirim sinyal ke proses menggunakan nama proses atau argumen *command-line*.

```bash
# Menghentikan proses berdasarkan nama parsial
pkill nginx
pkill -9 node

# Mencocokkan full command line / path (-f)
pkill -f "python app.py"
pkill -9 -f "worker.py"

# Menghentikan proses milik user tertentu (-u)
pkill -u www-data
pkill -9 -u john

# Menghentikan proses yang paling baru dibuat (-n) atau paling lama (-o)
pkill -n python
pkill -o python

# Case-insensitive matching (-i)
pkill -i firefox
```

---

## 5. Perintah `killall` (Berdasarkan Nama Program)

`killall` membunuh semua proses yang memiliki nama binary yang **persis sama**.

```bash
# Menghentikan semua proses bernama 'chrome'
killall chrome

# Memaksa berhenti semua proses 'apache2'
killall -9 apache2

# Meminta konfirmasi sebelum mematikan proses (-i / interactive)
killall -i firefox

# Menghentikan proses berdasarkan umur proses
killall -y 2h nginx    # Proses yang berjalan kurang dari 2 jam
killall -o 5h python   # Proses yang berjalan lebih dari 5 jam

# Menghentikan proses milik user tertentu
killall -u john python
```

---

## 6. Kill Berdasarkan Port / File (`fuser` & `lsof`)

Sangat berguna saat Anda mengalami error `Address already in use` (misal port 8080 terkunci).

### A. Menggunakan `fuser`
```bash
# Cek proses yang mendengarkan di port TCP 8080
fuser 8080/tcp

# Bunuh langsung proses di port 8080
fuser -k 8080/tcp

# Bunuh secara paksa (-9) di port 8080
fuser -k -9 8080/tcp
```

### B. Menggunakan `lsof` + `kill`
```bash
# Cek PID di port 3000
lsof -i :3000

# One-liner kill proses di port 3000
kill -9 $(lsof -t -i:3000)
```

---

## 7. Trik Praktis & One-Liners Populer

```bash
# 1. Kill semua proses Python yang sedang hang/berjalan
ps aux | grep '[p]ython' | awk '{print $2}' | xargs kill -9

# 2. Kill semua proses milik user tertentu (logout paksa user)
pkill -u nama_user -9

# 3. Kill proses yang memakan CPU tertinggi
kill -9 $(ps -eo pid,%cpu --sort=-%cpu | head -n 2 | tail -n 1 | awk '{print $1}')

# 4. Kill proses yang memakan Memori (RAM) tertinggi
kill -9 $(ps -eo pid,%mem --sort=-%mem | head -n 2 | tail -n 1 | awk '{print $1}')

# 5. GUI Kill (Klik jendela aplikasi yang freeze)
xkill
```

---

## 8. Best Practices & Troubleshooting

1. **Urutan Pemadaman yang Benar:**
   - **Langkah 1:** Selalu coba `kill <PID>` (`SIGTERM`) terlebih dahulu agar aplikasi sempat menutup koneksi database, menyimpan file, dan menghapus file temporary.
   - **Langkah 2:** Tunggu 5–10 detik.
   - **Langkah 3:** Gunakan `kill -9 <PID>` (`SIGKILL`) hanya jika aplikasi tidak merespon sama sekali (*hung / freeze*).

2. **Kenapa `kill -9` Gagal Membunuh Proses?**
   - **Zombie Process (`<defunct>`):** Proses sudah mati tetapi induknya belum membaca exit status. Zombie tidak memakan resource selain slot di tabel proses. Solusi: Bunuh proses *parent*-nya (`PPID`).
   - **Uninterruptible Sleep (`D` state):** Proses sedang menunggu operasi hardware I/O (misal disk failure, hanging NFS). Proses ini tidak dapat dibunuh sampai operasi I/O selesai atau sistem di-*reboot*.

3. **Memeriksa Parent Process ID (PPID):**
   ```bash
   ps -o pid,ppid,state,cmd -p <PID_ZOMBIE>
   # Bunuh parent process:
   kill -9 <PPID>
   ```

4. **Izin Akses (Permissions):**
   - User biasa hanya bisa mematikan proses miliknya sendiri.
   - Gunakan `sudo kill ...` untuk menghentikan proses milik sistem atau user lain (root).
