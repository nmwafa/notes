# Linux Server Hardening — *Debian based*

## Daftar isi:
- [Bagian 1: Persiapan](#bagian-1-persiapan)
- [Bagian 2: Keamanan Akses dan SSH](#bagian-2-keamanan-akses-dan-ssh)
  - [2.1 Ubah port standar](#21-ubah-port-standar)
  - [2.2 Jangan izinkan login dengan user root](#22-jangan-izinkan-login-dengan-user-root)
  - [2.3 Putus koneksi otomatis setelah tidak ada aktivitas](#23-putus-koneksi-otomatis-setelah-tidak-ada-aktivitas)
- [Bagian 3: Manajemen User & Hak Akses](#bagian-3-manajemen-user-dan-hak-akses)
  - [3.1 Hapus user atau grup tidak terpakai](#31-hapus-user-atau-grup-tidak-terpakai)
  - [3.2 Selalu gunakan sudo](#32-selalu-gunakan-sudo)
  - [3.3 Pastikan tidak ada user dengan id 0 dan group 0 selain root](#33-pastikan-tidak-ada-user-dengan-id-0-dan-grup-0-selain-root)
  - [3.4 Pastikan tidak ada user dengan password kosong](#34-pastikan-tidak-ada-user-dengan-password-kosong)
- [Bagian 4: Keamanan Jaringan](#bagian-4-keamanan-jaringan)
  - [4.1 Tutup semua port, kecuali yang dibutuhkan](#41-tutup-semua-port-kecuali-yang-dibutuhkan)
  - [4.2 Aktifkan UFW](#42-aktifkan-ufw)
  - [4.3 Pasang iptables](#43-pasang-iptables)
  - [4.4 Proteksi tambahan dengan fail2ban](#44-proteksi-tambahan-dengan-fail2ban)
  - [4.5 Matikan service tidak perlu](#45-matikan-service-tidak-perlu)
- [Bagian 5: Log, Audit dan Monitor](#bagian-5-log-audit-dan-monitor)
  - [5.1 Pakai lynis untuk audit keamanan sistem](#51-pakai-lynis-untuk-audit-keamanan-sistem)
  - [5.2 Pasang auditd](#52-pasang-auditd)
- [Bagian 6: Pasang Malware Scanner](#bagian-6-pasang-malware-scanner)
  - [6.1 maldet](#61-maldet)
  - [6.2 clamAV](#62-clamav)
  - [6.3 rkhunter](#63-rkhunter)
  - [6.4 chkrootkit](#64-chkrootkit)
- [Bagian 7: SELinux](#bagian-7-selinux)

---

## Bagian 1: Persiapan

Update Repositori & Package: 

```bash
sudo apt update && sudo apt upgrade -y
```

Cek versi kernel: 

```
uname -r
```

Cari apakah ada kerentanan di versi itu. Jika ada, segera upgrade.

---

## Bagian 2: Keamanan Akses dan SSH

### 2.1 Ubah port standar

> *Perubahan port disarankan untuk dilakukan secara lokal, bukan melalui remote karena berpotensi putus koneksi di tengah jalan ketika sedang di setting*
>
> *Sebelum merubah port ssh, pastikan dulu tidak ada lokal firewall atau jika ada lokal firewall dipastikan sudah ditambahkan policy allow port 23456*
>
> *Pastikan juga jika sebelumnya sudah menggunakan SELinux atau AppArmor sudah di update untuk allow port 23456*

```bash
sudo sed -i "s/#Port 22/Port 23456/" /etc/ssh/sshd_config
```

Cek listening port, pastikan port SSH sudah berubah: 

```bash
ss -tunlp | grep ssh

# Atau
netstat -tunlp | grep 23456
```

Jika belum, kemungkinan sistem pakai *Socket Activation*. Jalankan perintah:

```bash
sudo systemctl status ssh.socket
```

Jika statusnya *Active*, maka konfigurasi di *sshd_config* akan diabaikan karena *systemd* "memaksa" port 22 tetap berjalan

Beritahu *systemd* untuk pindah *port*. Jalankan perintah edit ini: 

```
sudo systemctl edit ssh.socket
```

Tambahkan atau edit baris berikut:

```
[Socket]
ListenStream=
ListenStream=23456
```

`ListenStream=` yang kosong pertama kali itu untuk menghapus *port* default 22

Muat ulang daemon dan restart socket:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket
sudo systemctl restart ssh
```

### 2.2 Jangan izinkan login dengan user root

```bash
sed -i "s/PermitRootLogin yes/PermitRootLogin no/" /etc/ssh/sshd_config
```

### 2.3 Putus koneksi otomatis setelah tidak ada aktivitas

```bash
sed -i "s/#ClientAliveInterval 0/ClientAliveInterval 300/" /etc/ssh/sshd_config` (300 detik x 3)
```

---

## Bagian 3: Manajemen User dan Hak Akses

### 3.1 Hapus user atau grup tidak terpakai

Cek di file `/etc/passwd` dan `/etc/group`

### 3.2 Selalu gunakan sudo

Untuk melakukan tugas administratif, pastikan selalu menggunakan sudo (tidak dijalankan langsung sebagai root)

### 3.3 Pastikan tidak ada user dengan id 0 dan group 0 selain root

```bash
awk -F: '($3 == 0) || ($4 == 0)' /etc/passwd
```

### 3.4 Pastikan tidak ada user dengan password kosong 

Cek di file `/etc/shadow`

---

## Bagian 4: Keamanan Jaringan

Cara mudah untuk cek apakah server sudah *compromised*:

```bash
lsof -i -nP | grep ESTABLISHED
```

Jika terlihat ada koneksi yang mencurigakan, segera tutup koneksi.

```bash
# kill [Process ID / PID]
# PID terlihat di kolom kedua pada output perintah sebelumnya

sudo kill 1234567
```

### 4.1 Tutup semua port, kecuali yang dibutuhkan

Cek port terbuka dengan: `ss -tunlp` atau `netstat -tunlp`

### 4.2 Aktifkan UFW

```bash
sudo systemctl enable --now ufw
```

Hanya izinkan koneksi masuk ke port yang dibutuhkan

```bash
ufw allow https
```

Proteksi dasar SSH

```bash
# Tolak IP yang coba login lebih dari 6x dalam 30 detik
sudo ufw limit ssh
```

### 4.3 Pasang iptables

Instalasi

```bash
sudo apt install iptables
```

Tolak koneksi berlebih untuk cegah DoS di port 80,443:

```bash
sudo iptables -I INPUT 1 -p tcp -m multiport --dports 80,443 -m connlimit --connlimit-above 20 --connlimit-mask 32 -j DROP
```

Izinkan koneksi loopback & yang sudah *established*:

```bash
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Izinkan akses standar:

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

Tolak yang lain: 

```bash
sudo iptables -P INPUT DROP
```

| Perintah | Deskripsi |
|----------|-----------|
| `sudo iptables -L -n` | Lihat *rules* standar |
| `sudo iptables -L -n -v` | Lihat *rules* detail statistik |
| `sudo iptables -L --line-numbers` | Lihat *rules* nomor baris |
| `sudo iptables -S` | Lihat *rules* format skrip |

### 4.4 Proteksi tambahan dengan fail2ban

Fungsinya untuk memproteksi terhadap serangan *bruteforce* dengan blokir IP

Instalasi

```bash
sudo apt install fail2ban
```

Buat salinan agar tidak tertimpa saat update: 

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Edit file `jail.local`:
    
```
[sshd]
enabled = true
port = ssh
bantime = 1d
findtime = 5m
maxretry = 3
backend = systemd -> jika pakai distro terbaru
```

Jika pakai ufw, tambahkan: `banaction = ufw` di bagian `[DEFAULT]`

Jika port SSH sudah diganti, sesuaikan di `/etc/services`

Cek status: 

```bash
fail2ban-client status sshd
```

### 4.5 Matikan service tidak perlu 

Cek: 

```bash
systemctl list-unit-files --state=enabled
```

---

## Bagian 5: Log, Audit dan Monitor

| Perintah | Deskripsi |
|----------|-----------|
| `who -H` | Lacak user yang login saat ini. membaca file `/var/log/utmp` |
| `last -R` | Lacak pengguna yang sebelumnya pernah login. membaca file `/var/log/wtmp` |
| `lastb` | Lacak upaya kegagalan login ke sistem. membaca file `/var/log/btmp` |

### 5.1 Pakai lynis untuk audit keamanan sistem

Instalasi

```bash
sudo apt install lynis
```

Jalankan

```bash
lynis audit system
```

### 5.2 Pasang auditd

Instalasi

```bash
sudo apt install auditd

# Aktifkan
sudo systemctl enable --now auditd
```

Memantau File atau Direktori: `auditctl -w [path_file] -p [izin] -k [label]`

```bash
sudo auditctl -w /etc/passwd -p wa -k ubah-password
```

Memantau Panggilan Sistem (*System Calls*):

```bash
sudo auditctl -a exit,always -F arch=b64 -S unlink -k hapus-file
```

Buat aturan permanen 

```bash
sudo nano /etc/audit/rules.d/audit.rules
```

Tambahkan atau edit:

```
-w /etc/shadow -p wa -k akses-shadow
-w /etc/ssh/sshd_config -p wa -k konfigurasi-ssh
```
    
Aktifkan perubahan: 

```bash
sudo augenrules --load
```

Log ada di /var/log/audit/audit.log

| Perintah | Deskripsi |
|----------|-----------|
| `sudo ausearch -k ubah-password` | Cari event |
| `sudo aureport -au` | Ringkasan laporan login |
| `sudo aureport -c --failed` | Laporan Perintah yang Gagal |
| `sudo aureport -f` | Ringkasan Akses File |

---

## Bagian 6: Pasang Malware Scanner

### 6.1 maldet

Instalasi

```bash
wget  http://www.rfxn.com/downloads/maldetect-current.tar.gz
tar xzf maldetect-current.tar.gz
cd maldetect-*
./install.sh
```

Konfigurasi

```bash
sudo nano /usr/local/maldetect/conf.maldet

# Tambahkan atau edit
quarantine_hits="1"
quarantine_clean="1"
```

Jalankan

```bash
maldet -u
maldet -a /var/www/html/
```

### 6.2 clamAV

```bash
# Instalasi
sudo apt install clamav clamav-daemon

# Update database
sudo freshclam

# Penggunaan
sudo clamscan -r /
```

### 6.3 rkhunter

```bash
# Instalasi
sudo apt install rkhunter

# Penggunaan
sudo rkhunter --update
sudo rkhunter --propupd
sudo rkhunter --check
```

### 6.4 chkrootkit

```bash
# Instalasi
sudo apt install chkrootkit

# Penggunaan
sudo chkrootkit
```

---

## Bagian 7: SELinux

Instalasi 

```bash
sudo apt install policycoreutils selinux-utils selinux-basics
```

Cek status 

```bash
sestatus
```

Aktifkan di Bootloader

```bash
sudo selinux-activate
```

Set ke mode *permissive* dulu sebelum reboot

```bash
sudo nano /etc/selinux/config
```

Edit -> `SELINUX=permissive`
 
Buat file penanda kernel saat proses booting

```bash
sudo touch /.autorelabel
```

Reboot

Cek port yang diizinkan

```bash
sudo semanage port -l
```

Daftarkan port kustom ke SELinux. e.g.,:

```bash
sudo semanage port -a 23456 -t ssh_port_t -p tcp
sudo semanage port -a 8000 -t http_port_t -p tcp
```

Jika port 8000 sudah digunakan oleh layanan lain dalam definisi SELinux, pakai `-m` (modify) sebagai ganti `-a`

Izinkan *Reverse Proxy*

```bash
sudo setbool -P httpd_can_network_connect 1
```

Jika muncul *error semanage command not found*, instal: 

```bash
sudo apt install policycoreutils-python-utils
```

Cek apakah ada proses yang hampir diblokir

```bash
sudo ausearch -m avc -ts recent
```

Jika tidak ada output atau tidak ada pesan *denied* yang kritikal, berarti konfigurasi sudah benar

Lihat mode saat ini

```bash
getenforce
```

Aktifkan mode *enforcing*:

  - Ubah mode secara instan tanpa reboot: `sudo setenforce 1`
  - Ubah permanen: `sudo nano /etc/selinux/config`, edit: `SELINUX=enforcing`
    
**[Penting]** Hati-hati sebelum di ubah ke mode `enforcing`. Pastikan SELinux tidak memblokir akses masuk seperti layanan SSH
