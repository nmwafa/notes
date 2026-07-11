# Checklist Pengujian Keamanan MFA/2FA

> Panduan pengujian keamanan untuk menemukan kelemahan pada implementasi Multi-Factor Authentication (MFA) / Two-Factor Authentication (2FA) dalam aplikasi web.
> 
> Dokumen ini ditujukan untuk security researcher, penetration tester, dan bug bounty hunter yang bekerja dalam lingkup pengujian yang **sah dan berizin**.
> 
> Seluruh teknik di bawah ini hanya boleh dilakukan pada sistem yang Anda miliki atau memiliki izin eksplisit tertulis untuk diuji. Penggunaan tanpa izin adalah tindakan ilegal.

---

## Daftar Isi

1. [Manipulasi Response](#1-manipulasi-response)
2. [Akses Halaman Langsung (Direct Request)](#2-akses-halaman-langsung-direct-request)
3. [Brute Force OTP](#3-brute-force-otp)
4. [Kode Cadangan (Backup Codes)](#4-kode-cadangan-backup-codes)
5. [Logika Verifikasi Cacat (Flawed Logic)](#5-logika-verifikasi-cacat-flawed-logic)
6. [Serangan Berbasis Waktu (Time-based Attacks)](#6-serangan-berbasis-waktu-time-based-attacks)
7. [Saluran Autentikasi Alternatif](#7-saluran-autentikasi-alternatif)
8. [Manajemen Sesi](#8-manajemen-sesi)
9. [Alur Pemulihan (Recovery Flow)](#9-alur-pemulihan-recovery-flow)
10. [Pendaftaran MFA (MFA Enrollment)](#10-pendaftaran-mfa-mfa-enrollment)
11. [Kebocoran Informasi (Information Leakage)](#11-kebocoran-informasi-information-leakage)
12. [CSRF pada Fitur MFA](#12-csrf-pada-fitur-mfa)
13. [Clickjacking pada Fitur MFA](#13-clickjacking-pada-fitur-mfa)
14. [Analisis File JavaScript](#14-analisis-file-javascript)
15. [Kelemahan Lainnya](#15-kelemahan-lainnya)

---

## 1. Manipulasi Response

> **Deskripsi:** Pengujian apakah server mempercayai input dari sisi klien untuk menentukan keberhasilan verifikasi 2FA. Kerentanan ini terjadi ketika logika validasi berada di sisi klien (front-end) alih-alih di server.

### 1.1 Manipulasi Body Response

- Kirim kode 2FA yang salah, lalu tangkap responsenya
- Cari nilai seperti `"success": false` atau `"status": "failed"` dalam response
- Ubah nilai tersebut menjadi `"success": true` menggunakan proxy (Burp Suite, OWASP ZAP)
- Forward request yang sudah dimodifikasi dan periksa apakah akses berhasil

```
Contoh response asli:
{ "success": false, "message": "Invalid OTP" }

Response yang dimanipulasi:
{ "success": true, "message": "Invalid OTP" }
```

**Dampak jika rentan:** Penyerang dapat melewati 2FA sepenuhnya tanpa mengetahui kode yang benar.

---

### 1.2 Manipulasi Status Code HTTP

- Periksa status code response pada saat verifikasi 2FA gagal (biasanya `401`, `403`, atau `4XX`)
- Ubah status code menjadi `200 OK` menggunakan intercept proxy
- Periksa apakah aplikasi menganggap autentikasi berhasil setelah perubahan ini

```
Response asli:    HTTP/1.1 401 Unauthorized
Response diubah:  HTTP/1.1 200 OK
```

**Dampak jika rentan:** Logika front-end yang bergantung pada status code dapat tertipu, membuka akses tanpa validasi server yang tepat.

---

### 1.3 Penghapusan Pesan Error

- Coba hapus seluruh body response error sebelum di-forward
- Uji apakah aplikasi masih berfungsi dengan response body kosong

> **Catatan Analisis:** Bagian ini tidak ada dalam dokumen asli. Ditambahkan karena beberapa aplikasi menggunakan kehadiran pesan error sebagai sinyal penolakan; tanpa pesan error, request dapat diteruskan.

---

## 2. Akses Halaman Langsung (Direct Request)

> **Deskripsi:** Pengujian apakah URL yang seharusnya hanya dapat diakses setelah melewati 2FA dapat diakses secara langsung tanpa menyelesaikan proses verifikasi.

### 2.1 Akses Langsung Tanpa 2FA

- Setelah memasukkan kredensial (username/password), catat URL yang dituju setelah 2FA berhasil
- Tanpa mengisi kode 2FA, navigasi langsung ke URL tujuan tersebut
- Periksa apakah server melakukan validasi sesi 2FA atau tidak

---

### 2.2 Manipulasi Header Referer

- Jika akses langsung ditolak, ubah header `Referer` agar seolah-olah request berasal dari halaman 2FA
- Contoh: `Referer: https://target.com/login/2fa`
- Forward request dan periksa apakah server terkecoh

```http
GET /dashboard HTTP/1.1
Host: target.com
Cookie: session=abc123
Referer: https://target.com/login/2fa
```

**Dampak jika rentan:** Server memvalidasi Referer (yang dapat dipalsukan) bukan status sesi 2FA yang sesungguhnya.

---

### 2.3 Pengujian Endpoint API Secara Terpisah

- Identifikasi endpoint API yang dipanggil oleh front-end
- Uji apakah endpoint API tersebut juga memerlukan 2FA atau hanya bergantung pada token/cookie sesi biasa
- Uji bagian-bagian aplikasi yang berbeda (admin panel, API v1 vs v2, subdomain, dll.)

> **Catatan Analisis:** Bagian pengujian endpoint API dan multi-bagian aplikasi ini tidak ada dalam dokumen asli. Penting karena banyak aplikasi menerapkan 2FA di front-end namun mengabaikannya di lapisan API.

---

## 3. Brute Force OTP

> **Deskripsi:** Pengujian apakah sistem membatasi jumlah percobaan memasukkan kode OTP yang salah. Tanpa pembatasan, penyerang dapat mencoba semua kemungkinan kombinasi kode.

### 3.1 Pengujian Rate Limiting

- Kirim request verifikasi 2FA sebanyak 100–200 kali berturut-turut dengan kode yang berbeda
- Periksa apakah ada pembatasan laju (rate limit) yang aktif
- Jika tidak ada batasan → **kerentanan rate limit**

---

### 3.2 Pengujian Account Lockout

- Masukkan kode 2FA yang salah secara berulang (10–20 kali)
- Periksa apakah akun dikunci sementara setelah sejumlah percobaan gagal
- Uji apakah lockout bisa di-bypass dengan mengganti IP atau memodifikasi header

> **Catatan Analisis:** Pengujian account lockout tidak ada secara eksplisit dalam dokumen asli. Ini merupakan kontrol keamanan penting yang perlu diverifikasi.

---

### 3.3 Panjang Kode OTP

- Identifikasi panjang kode OTP yang digunakan (4 digit vs 6 digit)
- Kode 4 digit = 10.000 kemungkinan (lebih mudah di-brute force)
- Kode 6 digit = 1.000.000 kemungkinan (lebih aman)
- Kode lebih pendek dari 6 digit harus dilaporkan sebagai temuan keamanan

---

### 3.4 Regenerasi OTP Tak Terbatas

- Coba minta kode OTP baru secara berulang-ulang tanpa batas
- Jika OTP hanya 4 digit dan dapat diregenerasi tak terbatas, lakukan:
  - Generate OTP terus-menerus di satu sisi
  - Brute force semua kemungkinan di sisi lain
  - Keduanya akan bertemu di tengah

---

### 3.5 Rate Limit Alur vs Rate Limit Kode

- Cek apakah rate limit hanya ada pada alur utama (flow), bukan pada verifikasi kode individual
- Coba brute force secara lambat (1 thread, delay antar request) untuk menghindari deteksi

---

### 3.6 Reset Rate Limit Melalui Resend OTP

- Masukkan kode yang salah beberapa kali hingga mendekati batas
- Minta pengiriman ulang kode OTP (`Resend OTP`)
- Periksa apakah hitungan percobaan gagal direset setelah resend

**Dampak jika rentan:** Rate limit dapat di-reset setiap kali OTP baru diminta, membuat brute force menjadi mungkin secara bertahap.

---

## 4. Kode Cadangan (Backup Codes)

> **Deskripsi:** Kode cadangan adalah mekanisme recovery ketika perangkat 2FA tidak tersedia. Jika tidak diamankan dengan benar, dapat menjadi jalur bypass 2FA.

### 4.1 Penerapan Single-Use (Sekali Pakai)

- Gunakan sebuah backup code untuk login
- Coba gunakan kembali backup code yang sama
- Jika berhasil digunakan lagi → **kerentanan kode cadangan dapat dipakai ulang**

---

### 4.2 Kekuatan Kode Cadangan

- Periksa panjang dan entropy backup code (minimal 8 karakter, alphanumerik acak)
- Kode yang terlalu pendek atau mudah ditebak harus dilaporkan

---

### 4.3 Penyimpanan Aman

- Uji apakah backup code ditampilkan dalam plain text di response API
- Periksa apakah backup code dapat diakses kembali setelah dibuat (seharusnya tidak)

---

### 4.4 Kontrol Akses pada Backup Codes

- Periksa apakah backup codes dapat diakses melalui endpoint yang tidak memerlukan autentikasi lengkap
- Coba akses endpoint pembuatan backup code secara langsung

**Catatan dari dokumen asli:** Backup codes seharusnya hanya tersedia pada satu permintaan segera setelah 2FA diaktifkan. Akses berikutnya seharusnya tidak menampilkan kode yang sama.

---

### 4.5 Terapkan Teknik 2FA pada Backup Codes

- Coba manipulasi response saat menggunakan backup code
- Coba manipulasi status code saat menggunakan backup code
- Coba brute force backup code (tergantung panjang dan entropy-nya)

---

## 5. Logika Verifikasi Cacat (Flawed Logic)

> **Deskripsi:** Kerentanan di mana server tidak memvalidasi bahwa pengguna yang melakukan langkah kedua (verifikasi kode) adalah pengguna yang sama yang menyelesaikan langkah pertama (login password).

### 5.1 Manipulasi Cookie Akun

- Login dengan akun milik sendiri dan tangkap cookie sesi yang ditetapkan
- Identifikasi apakah ada cookie yang berisi identifier akun (contoh: `account=attacker`)
- Ubah nilai cookie tersebut menjadi identifier akun korban (`account=victim`)
- Kirim kode 2FA yang valid (milik akun penyerang) dengan cookie yang sudah diubah

```http
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=victim-user    ← Diubah dari "attacker" ke "victim"

verification-code=123456       ← Kode valid milik attacker
```

**Dampak jika rentan:** Penyerang dapat mengakses akun korban menggunakan kode 2FA miliknya sendiri.

---

### 5.2 Berbagi Token Antar Akun (Missing 2FA Code Integrity Validation)

- Minta kode 2FA dari akun penyerang
- Gunakan kode tersebut pada request verifikasi akun korban
- Periksa apakah server memvalidasi bahwa kode 2FA benar-benar milik akun yang sedang diautentikasi

---

### 5.3 Penggunaan Ulang Token Sebelumnya

- Simpan token 2FA yang pernah berhasil digunakan
- Coba gunakan kembali token tersebut untuk sesi baru

---

## 6. Serangan Berbasis Waktu (Time-based Attacks)

> **Deskripsi:** OTP berbasis waktu (TOTP) memiliki jendela validitas terbatas. Kelemahan implementasi pada aspek waktu dapat menciptakan celah keamanan.

### 6.1 Toleransi Clock Skew

- Uji apakah server menerima kode OTP yang sudah kadaluwarsa beberapa menit (clock skew)
- Toleransi yang terlalu besar (lebih dari 30 detik) dapat memperluas jendela serangan brute force

> **Catatan Analisis:** Bagian clock skew tidak ada dalam dokumen asli. Penting untuk diuji terutama pada implementasi TOTP (RFC 6238).

---

### 6.2 Penggunaan Ulang Kode dalam Jendela Waktu

- Gunakan kode OTP yang valid
- Segera coba gunakan kode yang sama lagi dalam jendela waktu yang masih aktif
- Server yang baik harus menolak penggunaan ulang kode meski masih valid secara waktu

---

### 6.3 Penerimaan Kode Kadaluwarsa

- Minta kode OTP namun tunda penggunaannya selama >5 menit
- Coba gunakan kode tersebut setelah seharusnya kadaluwarsa
- Coba kembali setelah durasi yang sangat lama (1 hari)

**Catatan dari dokumen asli:** Durasi 1 hari cukup untuk melakukan cracking/guessing kode 6 digit, sehingga kode yang tidak kadaluwarsa merupakan kerentanan serius.

---

## 7. Saluran Autentikasi Alternatif

> **Deskripsi:** Aplikasi modern sering memiliki beberapa jalur akses (web, API, mobile, SSO). Kelemahan umum adalah ketika 2FA hanya diterapkan di satu jalur namun tidak di jalur lainnya.

### 7.1 API Melewati 2FA

- Identifikasi versi API yang berbeda (`/v1/`, `/v2/`, `/v3/`)
- Uji apakah endpoint API lama masih aktif dan tidak memerlukan 2FA
- Uji endpoint API yang digunakan oleh mobile app

---

### 7.2 Mobile App Melewati 2FA

- Jika ada aplikasi mobile, uji apakah alur autentikasi mobile menerapkan 2FA
- Bandingkan flow autentikasi web vs mobile

> **Catatan Analisis:** Bagian ini tidak ada dalam dokumen asli. Kritis karena banyak aplikasi menerapkan 2FA di web namun melupakan implementasinya di mobile app.

---

### 7.3 SSO Melewati 2FA

- Jika aplikasi mendukung Single Sign-On (SSO), uji apakah login via SSO melewati 2FA
- Identifikasi apakah provider SSO (Google, GitHub, dll.) memiliki 2FA sendiri yang dipercaya

---

### 7.4 Subdomain dengan Versi Lama

- Cari subdomain testing/staging (contoh: `test.target.com`, `staging.target.com`)
- Subdomain ini sering menggunakan versi lama yang belum menerapkan 2FA

---

### 7.5 Manipulasi Mode MFA

Berdasarkan kasus nyata (Grammarly):
- Tangkap request verifikasi 2FA
- Ubah parameter `"mode": "sms"` menjadi `"mode": "email"`
- Ubah parameter `"secureLogin": true` menjadi `"secureLogin": false`
- Periksa apakah perubahan mode memungkinkan bypass verifikasi

---

## 8. Manajemen Sesi

> **Deskripsi:** Sesi yang dibuat sebelum atau selama proses 2FA harus dikelola dengan benar. Sesi yang tidak diinvalidasi atau dapat diprediksi menciptakan celah keamanan.

### 8.1 Sesi Sebelum 2FA Tidak Dibatasi

- Login ke akun yang sama di dua browser/device yang berbeda
- Aktifkan 2FA di browser pertama
- Periksa apakah sesi di browser kedua masih aktif tanpa perlu melewati 2FA
- Jika aktif → celah keamanan karena sesi yang dibajak sebelum aktivasi 2FA tetap bisa digunakan

---

### 8.2 Sesi Sebelum MFA Masih Valid Setelah Aktivasi MFA

- Buat sesi di device A
- Aktifkan MFA dari device B
- Kembali ke device A dan reload halaman
- Jika sesi masih aktif tanpa diminta MFA → **insufficient session expiration**

---

### 8.3 Session Fixation dengan MFA

- Uji apakah session ID diperbarui setelah berhasil menyelesaikan 2FA
- Session ID yang sama sebelum dan sesudah 2FA dapat rentan terhadap session fixation

> **Catatan Analisis:** Bagian ini tidak ada dalam dokumen asli. Session fixation + 2FA adalah kombinasi kerentanan yang perlu diuji secara spesifik.

---

### 8.4 Izin Sesi (Session Permission)

- Buka sesi untuk akun penyerang dan akun korban secara bersamaan
- Selesaikan 2FA pada akun penyerang
- Coba akses langkah berikutnya menggunakan sesi akun korban
- Jika server hanya memeriksa boolean `"2FA_passed": true` dalam sesi tanpa mengaitkannya ke akun → kerentanan

---

### 8.5 Fungsi "Remember This Device"

- Identifikasi mekanisme "ingat perangkat ini" (biasanya cookie atau token)
- Periksa apakah cookie yang digunakan dapat ditebak/di-brute force
- Periksa apakah fungsi ini terikat ke IP address (dapat dipalsukan dengan header `X-Forwarded-For`)
- Uji apakah cookie "remember device" dapat digunakan dari device lain

---

## 9. Alur Pemulihan (Recovery Flow)

> **Deskripsi:** Fitur pemulihan akun (reset password, ubah email) sering menjadi jalur alternatif yang tidak memerlukan 2FA, sehingga menjadi target serangan.

### 9.1 Reset Password Melewati 2FA

- Lakukan reset password untuk akun yang memiliki 2FA
- Setelah reset berhasil, cek apakah akun langsung login tanpa diminta 2FA
- Cek apakah 2FA otomatis dinonaktifkan setelah reset password

**Catatan dari dokumen asli:** Hampir semua aplikasi web secara otomatis login pengguna setelah berhasil mereset password. Ini bisa menjadi bypass 2FA yang efektif.

---

### 9.2 Perubahan Email Menonaktifkan 2FA

- Ubah alamat email akun
- Periksa apakah 2FA otomatis dinonaktifkan setelah perubahan email
- Ini bergantung pada kebijakan organisasi namun berpotensi menjadi masalah

---

### 9.3 Link Reset Password yang Dapat Digunakan Ulang

- Minta link reset password
- Gunakan link tersebut untuk reset
- Coba gunakan link yang sama lagi untuk login ulang

---

### 9.4 Ketahanan terhadap Social Engineering

- Evaluasi apakah alur pemulihan cukup sulit untuk dipalsukan oleh pihak ketiga
- Apakah pertanyaan keamanan atau verifikasi identitas cukup kuat?

> **Catatan Analisis:** Aspek social engineering resistance tidak ada dalam dokumen asli namun kritis dalam konteks recovery flow.

---

## 10. Pendaftaran MFA (MFA Enrollment)

> **Deskripsi:** Proses mendaftarkan metode 2FA baru harus dilindungi dengan baik, karena kelemahan di sini dapat memungkinkan penyerang mendaftarkan perangkat 2FA miliknya ke akun korban.

### 10.1 Melewati Proses Pendaftaran MFA

- Periksa apakah pengguna dapat melewati langkah aktivasi 2FA saat registrasi akun
- Uji apakah ada parameter URL atau cookie yang dapat dimanipulasi untuk melompati langkah ini

---

### 10.2 Menonaktifkan 2FA Tanpa Autentikasi yang Memadai

- Coba nonaktifkan 2FA tanpa memasukkan password yang benar
- Coba nonaktifkan 2FA hanya dengan kode backup tanpa password

**Kasus nyata dari dokumen asli:**
```
1. Aktifkan 2FA dari /settings/auth
2. Klik tombol nonaktifkan 2FA
3. Masukkan kode autentikasi yang valid
4. Di kolom password, masukkan nilai sembarang
5. Jika 2FA berhasil dinonaktifkan → password tidak divalidasi
```

---

### 10.3 Aktivasi 2FA Tanpa Verifikasi Email

- Daftar akun menggunakan email korban (belum diverifikasi)
- Login tanpa memverifikasi email
- Aktifkan 2FA pada akun tersebut
- **Dampak:** Penyerang dapat mengunci korban dari akun mereka sendiri

---

### 10.4 Proteksi Re-enrollment

- Setelah 2FA aktif, coba daftarkan perangkat 2FA baru tanpa menonaktifkan yang lama
- Apakah proses re-enrollment memerlukan verifikasi dengan 2FA yang sudah ada?

> **Catatan Analisis:** Bagian ini tidak ada dalam dokumen asli. Re-enrollment tanpa validasi memungkinkan account takeover jika session dikompromikan.

---

## 11. Kebocoran Informasi (Information Leakage)

> **Deskripsi:** Aplikasi yang tidak sengaja mengekspos kode 2FA atau informasi sensitif lainnya dalam response HTTP.

### 11.1 Kebocoran Kode OTP dalam Response

- Tangkap request yang memicu pengiriman OTP (misal: klik "Send OTP")
- Periksa body response dari request tersebut
- Kode OTP yang muncul dalam response → **kerentanan serius**

---

### 11.2 Informasi Sensitif di Halaman 2FA

- Periksa apakah halaman 2FA menampilkan informasi yang tidak seharusnya terlihat
- Contoh: sebagian nomor telepon yang tidak di-mask, nama lengkap, dll.

---

### 11.3 Analisis File JavaScript

- Saat memicu request kode 2FA, buka semua file JS yang direferensikan dalam response
- Cari dalam file JS:
  - Kode OTP yang di-hardcode
  - Secret key TOTP
  - Logika validasi yang dapat dieksploitasi
  - Endpoint API tersembunyi

---

## 12. CSRF pada Fitur MFA

> **Deskripsi:** Cross-Site Request Forgery pada fitur terkait MFA dapat memungkinkan penyerang menonaktifkan atau memodifikasi pengaturan 2FA korban tanpa sepengetahuan mereka.

### 12.1 CSRF pada Fitur Nonaktifkan 2FA

- Buka fitur nonaktifkan 2FA dan tangkap request-nya
- Periksa apakah request mengandung CSRF token
- Coba kirim request tanpa CSRF token atau dengan token yang tidak valid
- Buat halaman HTML eksternal yang memicu request ini secara otomatis
- Jika berhasil menonaktifkan 2FA dari sumber eksternal → **kerentanan CSRF**

---

### 12.2 CSRF pada Fitur Ubah Pengaturan 2FA

- Uji CSRF pada seluruh endpoint yang berkaitan dengan konfigurasi 2FA:
  - Ubah metode 2FA
  - Tambah nomor telepon/email baru
  - Generate ulang backup codes

---

## 13. Clickjacking pada Fitur MFA

> **Deskripsi:** Clickjacking memungkinkan penyerang menyisipkan halaman aplikasi dalam iframe tersembunyi, kemudian mengelabui pengguna untuk mengklik tombol (seperti "Nonaktifkan 2FA") tanpa menyadarinya.

### 13.1 Pengujian Iframe pada Halaman Nonaktifkan 2FA

- Coba muat halaman penonaktifan 2FA dalam sebuah `<iframe>`:

```html
<iframe src="https://target.com/settings/disable-2fa"></iframe>
```

- Jika halaman berhasil dimuat dalam iframe → server tidak mengirim header `X-Frame-Options` atau `Content-Security-Policy: frame-ancestors`
- Jika berhasil, coba buat proof-of-concept serangan clickjacking

---

### 13.2 Header Keamanan

- Periksa response header pada halaman sensitif MFA:
  - `X-Frame-Options: DENY` atau `SAMEORIGIN`
  - `Content-Security-Policy: frame-ancestors 'self'`
- Ketiadaan header ini = potensi clickjacking

---

## 14. Analisis File JavaScript

> **Deskripsi:** File JavaScript yang digunakan oleh halaman 2FA dapat mengandung informasi sensitif atau logika yang dapat dieksploitasi.

### 14.1 Pencarian Endpoint Tersembunyi

- Kumpulkan semua file JS yang dimuat saat proses 2FA berlangsung
- Gunakan tools seperti `linkfinder`, `js-beautifier`, atau analisis manual
- Cari pola URL, token, atau fungsi validasi

---

### 14.2 Client-Side Rate Limit Bypass

- Identifikasi apakah rate limiting diimplementasikan di sisi klien (JavaScript)
- Jika ya, bypass dengan mengirim request langsung ke endpoint tanpa melalui logika JS

**Catatan dari dokumen asli:** Merujuk ke `rate-limit-bypass.md` — pastikan untuk menguji semua teknik rate limit bypass yang relevan.

---

## 15. Kelemahan Lainnya

### 15.1 Bypass dengan Kode Null atau 000000

- Coba masukkan `null`, `undefined`, `""` (string kosong), atau `000000` sebagai kode 2FA
- Beberapa implementasi buruk menerima nilai-nilai ini sebagai valid

---

### 15.2 Bypass dengan Mengirim Kode Kosong

Berdasarkan kasus nyata (Glassdoor):
- Tangkap request POST saat mengirim kode 2FA yang salah
- Hapus nilai kode dari parameter request (bukan set ke kosong, tapi hapus parameter-nya)
- Forward request dan periksa apakah berhasil login

```
Request original:  verification-code=123456
Request dimodifikasi: [parameter dihapus sepenuhnya]
```

---

### 15.3 Penyalahgunaan Fungsi Reset Password untuk Mengeksploitasi Sesi

Berdasarkan kasus nyata (Grammarly):
- Login ke akun yang sama di dua device berbeda
- Di device A, aktifkan 2FA
- Di device B, reload halaman
- Periksa apakah sesi di device B masih aktif tanpa diminta 2FA

---

## Referensi

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Testing Guide - Testing for Multi-Factor Authentication](https://owasp.org/www-project-web-security-testing-guide/)
- [RFC 6238 - TOTP: Time-Based One-Time Password Algorithm](https://datatracker.ietf.org/doc/html/rfc6238)
- [PortSwigger Web Security Academy - Authentication](https://portswigger.net/web-security/authentication)
