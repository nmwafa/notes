---
title: "Linux File Permissions"
layout: default
---

# Linux File Permissions

---

## Daftar Isi
1. [Model Kepemilikan & Struktur Izin Dasar](#1-model-kepemilikan--struktur-izin-dasar)
2. [Standar Linux Permissions (Read, Write, Execute)](#2-standar-linux-permissions-read-write-execute)
3. [Representasi Izin: Simbolik vs Oktal](#3-representasi-izin-simbolik-vs-oktal)
4. [Special Permissions (SUID, SGID, Sticky Bit)](#4-special-permissions-suid-sgid-sticky-bit)
5. [Umask (Default File Creation Mask)](#5-umask-default-file-creation-mask)
6. [POSIX Access Control Lists (ACL)](#6-posix-access-control-lists-acl)
7. [Atribut Berkas Khusus (File Attributes via chattr)](#7-atribut-berkas-khusus-file-attributes-via-chattr)
8. [Manajemen Izin via CLI (chmod, chown, chgrp)](#8-manajemen-izin-via-cli-chmod-chown-chgrp)
9. [Praktik Terbaik & Aspek Keamanan](#9-praktik-terbaik--aspek-keamanan)

---

## 1. Model Kepemilikan & Struktur Izin Dasar

Di Linux, setiap file dan direktori diasosiasikan dengan tiga entitas pemilik (*Ownership Triad*):

1. **User (Owner - `u`):** Akun pengguna yang memiliki berkas tersebut.
2. **Group (`g`):** Grup pengguna yang memiliki hak akses kolektif terhadap berkas.
3. **Others (`o`):** Semua pengguna lain di sistem yang bukan *owner* maupun anggota dari *group* berkas.

### Anatomi Output `ls -l`

Contoh output: `-rwxr-xr-- 1 wafa developers 4096 Aug 20 09:00 deploy.sh`

```text
-  rwx  r-x  r--  1  wafa  developers  4096  Aug 20 09:00  deploy.sh
│  ───  ───  ───
│   │    │    │
│   │    │    └─ Permissions untuk Others (o): Read-only
│   │    └────── Permissions untuk Group (g): Read & Execute
│   └─────────── Permissions untuk User/Owner (u): Read, Write, Execute
└─────────────── File Type: (-) Reguler, (d) Direktori, (l) Symlink, dll.
```

---

## 2. Standar Linux Permissions (Read, Write, Execute)

Arti dari hak akses standar berbeda tergantung pada tipe objeknya:

| Izin | Karakter | Nilai Oktal | Arti pada Berkas (File) | Arti pada Direktori (Folder) |
| :---: | :---: | :---: | :--- | :--- |
| **Read** | `r` | `4` | Membaca / melihat isi file (misal via `cat`, `less`). | Melihat daftar file di dalam direktori (`ls`). |
| **Write** | `w` | `2` | Mengubah / menimpa / memodifikasi isi file. | Membuat, menghapus, atau mengganti nama file di dalam direktori (*). |
| **Execute** | `x` | `1` | Menjalankan file sebagai program atau skrip biner. | Memasuki / menelusuri direktori (`cd`) dan mengakses metadata file di dalamnya. |

> **(*) Catatan Penting Mengenai Penghapusan File:**  
> Kemampuan menghapus atau me-rename file ditentukan oleh izin **Write (`w`) pada direktori induknya**, bukan izin write pada file itu sendiri!

---

## 3. Representasi Izin: Simbolik vs Oktal

### 1. Representasi Oktal (Numerik)
Setiap triplet izin dihitung dari jumlah bobot binernya:
- `r` = 4 (2^2)
- `w` = 2 (2^1)
- `x` = 1 (2^0)
- `-` = 0

Contoh perhitungan:
- `rwx` = 4 + 2 + 1 = 7
- `rw-` = 4 + 2 + 0 = 6
- `r-x` = 4 + 0 + 1 = 5
- `r--` = 4 + 0 + 0 = 4

**Contoh Format 3-Digit:**
- `chmod 755 file.sh` -> `rwxr-xr-x` (Owner: full, Group: read/exec, Others: read/exec)
- `chmod 644 file.txt` -> `rw-r--r--` (Owner: read/write, Group: read, Others: read)
- `chmod 700 secret/` -> `rwx------` (Owner: full, Group: none, Others: none)

---

## 4. Special Permissions (SUID, SGID, Sticky Bit)

Selain triplet standar 3-digit, Linux memiliki **digit ke-4** (paling depan) yang merepresentasikan izin khusus.

| Special Permission | Nilai Oktal | Letak Simbolik | Huruf Aktif / Nonaktif | Berlaku Pada |
| :--- | :---: | :---: | :---: | :--- |
| **SUID (Set User ID)** | `4` | Kolom execute User (`u`) | `s` (jika `x` aktif) / `S` (jika `x` mati) | Executable File |
| **SGID (Set Group ID)** | `2` | Kolom execute Group (`g`) | `s` (jika `x` aktif) / `S` (jika `x` mati) | File & Direktori |
| **Sticky Bit** | `1` | Kolom execute Others (`o`) | `t` (jika `x` aktif) / `T` (jika `x` mati) | Direktori |

---

### A. SUID (Set Owner User ID up on execution)
- **Fungsi:** Saat binary dieksekusi, proses berjalan dengan hak istimewa **pemilik file (Owner)**, bukan pengguna yang mengeksekusinya.
- **Contoh Umum:** `/usr/bin/passwd`. Pengguna biasa memerlukan akses menulis ke `/etc/shadow` saat mengganti password; dengan bit SUID, `passwd` berjalan sementara sebagai `root`.
- **Konfigurasi:**
  ```bash
  chmod u+s /path/to/binary
  chmod 4755 /path/to/binary
  ```

---

### B. SGID (Set Group ID)
- **Pada File Eksekusi:** Proses berjalan dengan hak istimewa dari **Group pemilik file**.
- **Pada Direktori (Shared Directory):** Setiap file atau subdirektori baru yang dibuat di dalam folder tersebut akan **mewarisi Group dari folder induk**, bukan group primer dari pengguna yang membuatnya. Sangat penting untuk folder kolaborasi tim.
- **Konfigurasi:**
  ```bash
  chmod g+s /var/shared/
  chmod 2775 /var/shared/
  ```

---

### C. Sticky Bit
- **Fungsi:** Diterapkan pada direktori publik (seperti `/tmp`). Mencegah pengguna menghapus atau me-rename file milik orang lain, meskipun mereka memiliki izin `write` pada direktori tersebut.
- **Aturan:** Hanya **Owner file**, **Owner direktori**, atau **root** yang dapat menghapus file tersebut.
- **Konfigurasi:**
  ```bash
  chmod +t /shared/public_folder
  chmod 1777 /shared/public_folder
  ```

---

## 5. Umask (Default File Creation Mask)

`umask` (*user mask*) menentukan izin default yang akan **dikurangkan / dimatikan** saat sebuah file atau direktori baru dibuat.

### Batas Izin Maksimum Sistem:

- File biasa: `666` (`rw-rw-rw-`) — sistem operasi sengaja tidak memberi bit eksekusi `x` pada file baru demi keamanan.
- Direktori: `777` (`rwxrwxrwx`).

### Logika Perhitungan:

#### Izin Akhir File = 666 - Umask
#### Izin Akhir Direktori = 777 - Umask

| Nilai Umask | Izin File Default | Izin Folder Default | Deskripsi |
| :---: | :---: | :---: | :--- |
| `0022` | `644` (`rw-r--r--`) | `755` (`rwxr-xr-x`) | Standar user biasa pada mayoritas distro. |
| `0027` | `640` (`rw-r-----`) | `750` (`rwxr-x---`) | Aman untuk server (others dilarang akses sama sekali). |
| `0077` | `600` (`rw-------`) | `700` (`rwx------`) | Sangat privat (hanya owner yang punya akses). |

```bash
# Melihat umask saat ini
umask

# Mengubah umask untuk sesi saat ini
umask 027
```

---

## 6. POSIX Access Control Lists (ACL)

Model izin standar UNIX terbatas pada satu *User*, satu *Group*, dan *Others*. Jika Anda ingin memberikan izin khusus ke *User B* atau *Group C* tanpa mengubah kepemilikan utama, gunakan **POSIX ACL**.

Tanda indikator: Berkas dengan ACL memiliki tanda plus (`+`) di akhir mode izin pada `ls -l` (misal: `-rw-rwxr--+`).

### Perintah Utama ACL:

```bash
# 1. Melihat entri ACL
getfacl /path/to/data

# 2. Memberikan hak Read & Write ke user tertentu (developer1)
setfacl -m u:developer1:rw /path/to/data

# 3. Memberikan hak ke group tertentu (audit_team)
setfacl -m g:audit_team:r-x /path/to/data

# 4. Mengatur Default ACL (diwariskan otomatis ke file/subfolder baru)
setfacl -d -m g:developers:rwx /var/shared_project

# 5. Menghapus entri ACL untuk user tertentu
setfacl -x u:developer1 /path/to/data

# 6. Menghapus seluruh entri extended ACL
setfacl -b /path/to/data
```

---

## 7. Atribut Berkas Khusus (File Attributes via chattr)

Linux filesystem (ext4, xfs, btrfs) mendukung atribut berkas level kernel yang dapat mengunci file bahkan dari akses user `root`.

```bash
# Melihat atribut berkas
lsattr /path/to/file

# 1. Atribut Immutable (+i): File tidak bisa diubah, dihapus, di-rename, di-link (bahkan oleh root)
sudo chattr +i /etc/resolv.conf

# Melepas atribut immutable
sudo chattr -i /etc/resolv.conf

# 2. Atribut Append-Only (+a): Hanya bisa menambah data di akhir file (ideal untuk log)
sudo chattr +a /var/log/custom_audit.log
```

---

## 8. Manajemen Izin via CLI (chmod, chown, chgrp)

### 1. Mengubah Izin (`chmod`)
```bash
# Notasi Simbolik: [ugoa] [+ - =] [rwxst]
chmod u+x script.sh              # Beri hak eksekusi ke owner
chmod g-w,o-r file.txt           # Cabut write group dan read others
chmod u=rw,go=r document.pdf     # Set spesifik owner rw, group/others r
chmod -R 750 /var/www/html       # Rekursif ke seluruh subfolder & file

# Notasi Oktal
chmod 755 /usr/local/bin/tools
chmod 4750 /usr/bin/custom_suid
```

### 2. Mengubah Pemilik & Grup (`chown` & `chgrp`)
```bash
# Mengubah owner
chown wafa report.pdf

# Mengubah owner sekaligus group
chown wafa:developers /var/project

# Mengubah owner secara rekursif
chown -R wafa:www-data /var/www/html

# Hanya mengubah group
chgrp developers /var/project
```

---

## 9. Praktik Terbaik & Aspek Keamanan

1. **Audit Biner SUID/SGID Secara Rutin:**  
   Biner SUID milik root adalah target utama eksploitasi eskalasi hak akses (*Privilege Escalation* via GTFOBins).
   ```bash
   # Mencari semua file dengan bit SUID di sistem:
   find / -perm -4000 -type f -exec ls -ld {} \; 2>/dev/null
   ```
2. **Jangan Pernah Menggunakan `chmod 777`:**  
   Gunakan perizinan spesifik (*Principle of Least Privilege*). Jika membutuhkan kolaborasi multi-user, manfaatkan **SGID pada direktori** atau **POSIX ACL**.
3. **Amankan Direktori Web Server:**  
   File konfigurasi atau skrip web harus dimiliki oleh user pengembang/root dengan izin `644`, sedangkan folder upload media dibatasi eksekusinya.
4. **Gunakan Umask yang Ketat untuk Server Produksi:**  
   Terapkan `umask 027` pada profile sistem (`/etc/profile` atau `/etc/login.defs`) untuk membatasi akses default *others*.
