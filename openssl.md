---
title: "OpenSSL Cheatsheet"
layout: default
---

# OpenSSL Cheatsheet

## Daftar Isi

1. [Cek Versi & Bantuan](#1-cek-versi--bantuan)
2. [Membuat Private Key](#2-membuat-private-key)
3. [Membuat Kunci Publik dari Private Key](#3-membuat-kunci-publik-dari-private-key)
4. [Membuat CSR (Certificate Signing Request)](#4-membuat-csr-certificate-signing-request)
5. [Membuat Sertifikat Self-Signed](#5-membuat-sertifikat-self-signed)
6. [Melihat Informasi Sertifikat / Key / CSR](#6-melihat-informasi-sertifikat--key--csr)
7. [Verifikasi Sertifikat & Chain](#7-verifikasi-sertifikat--chain)
8. [Test Koneksi SSL/TLS (s_client & s_server)](#8-test-koneksi-ssltls-s_client--s_server)
9. [Konversi Format Sertifikat (PEM, DER, PFX/P12)](#9-konversi-format-sertifikat-pem-der-pfxp12)
10. [Hashing / Digest (MD5, SHA256, dll)](#10-hashing--digest-md5-sha256-dll)
11. [Enkripsi & Dekripsi File Simetris](#11-enkripsi--dekripsi-file-simetris)
12. [Enkripsi & Dekripsi Asimetris (RSA)](#12-enkripsi--dekripsi-asimetris-rsa)
13. [Encoding Base64](#13-encoding-base64)
14. [Generate Angka Acak & Password](#14-generate-angka-acak--password)
15. [Membuat CA Sendiri & Menandatangani CSR](#15-membuat-ca-sendiri--menandatangani-csr)
16. [HMAC](#16-hmac)
17. [Diffie-Hellman Parameters](#17-diffie-hellman-parameters)
18. [Certificate Revocation List (CRL)](#18-certificate-revocation-list-crl)
19. [Benchmark Performa (speed)](#19-benchmark-performa-speed)
20. [Tips & Troubleshooting Umum](#20-tips--troubleshooting-umum)

## 1. Cek Versi & Bantuan

```bash
# Cek versi OpenSSL yang terpasang
openssl version

# Cek versi lebih detail (build flags, tanggal, dll)
openssl version -a

# Melihat daftar semua sub-command yang tersedia
openssl help

# Melihat opsi dari sub-command tertentu, misal x509
openssl x509 -help
```

## 2. Membuat Private Key

Perintah ini paling sering dipakai sebagai langkah pertama sebelum membuat CSR/sertifikat.

```bash
# RSA 2048-bit (standar umum)
openssl genrsa -out private.key 2048

# RSA 4096-bit (lebih kuat, lebih lambat)
openssl genrsa -out private.key 4096

# RSA dengan enkripsi passphrase (AES-256)
openssl genrsa -aes256 -out private.key 2048

# Key modern menggunakan Elliptic Curve (lebih ringan dari RSA)
openssl ecparam -name prime256v1 -genkey -noout -out ec-private.key

# Cara baru (OpenSSL 3.x) yang lebih generik dengan genpkey
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out private.key
openssl genpkey -algorithm ED25519 -out ed25519.key
```

## 3. Membuat Kunci Publik dari Private Key

Kunci publik biasanya diturunkan (di-*derive*) dari private key yang sudah ada, bukan dibuat terpisah dari nol.

```bash
# Extract public key dari RSA private key
openssl rsa -in private.key -pubout -out public.key

# Extract public key dari EC private key
openssl ec -in ec-private.key -pubout -out ec-public.key

# Cara generik OpenSSL 3.x yang berlaku untuk semua jenis algoritma (RSA/EC/ED25519/dst)
openssl pkey -in private.key -pubout -out public.key

# Jika private key terenkripsi dengan passphrase, tinggal tambahkan -passin
openssl rsa -in private.key -passin pass:passwordku -pubout -out public.key

# Melihat isi/detail dari public key
openssl rsa -pubin -in public.key -text -noout

# Membandingkan public key hasil extract dengan public key di sertifikat (harus identik)
openssl x509 -in cert.pem -pubkey -noout > public_from_cert.key
diff public.key public_from_cert.key
```

## 4. Membuat CSR (Certificate Signing Request)

CSR dibutuhkan saat ingin meminta sertifikat ke CA (misalnya untuk domain publik).

```bash
# Membuat CSR interaktif (akan ditanya Country, Organization, Common Name, dll)
openssl req -new -key private.key -out request.csr

# Membuat CSR langsung tanpa interaktif (subject di-inline)
openssl req -new -key private.key -out request.csr \
  -subj "/C=ID/ST=Jawa Tengah/L=Surakarta/O=Perusahaan Saya/OU=IT/CN=example.com"

# Membuat CSR dengan Subject Alternative Name (SAN) - wajib untuk browser modern
openssl req -new -key private.key -out request.csr \
  -subj "/CN=example.com" \
  -addext "subjectAltName=DNS:example.com,DNS:www.example.com"
```

## 5. Membuat Sertifikat Self-Signed

Berguna untuk keperluan development/testing tanpa perlu CA eksternal.

```bash
# Generate key + sertifikat self-signed sekaligus (valid 365 hari)
openssl req -x509 -newkey rsa:2048 -keyout private.key -out cert.pem \
  -days 365 -nodes -subj "/CN=localhost"

# Self-signed dengan SAN (agar tidak warning di browser saat testing localhost)
openssl req -x509 -newkey rsa:2048 -keyout private.key -out cert.pem \
  -days 365 -nodes -subj "/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"

# Membuat sertifikat self-signed dari CSR yang sudah ada
openssl x509 -req -in request.csr -signkey private.key -out cert.pem -days 365
```

## 6. Melihat Informasi Sertifikat / Key / CSR

Perintah untuk inspeksi ini sangat sering dipakai saat debugging masalah SSL.

```bash
# Melihat detail lengkap sertifikat (issuer, validity, SAN, dll)
openssl x509 -in cert.pem -text -noout

# Melihat tanggal kedaluwarsa sertifikat saja
openssl x509 -in cert.pem -noout -enddate

# Melihat subject dan issuer saja
openssl x509 -in cert.pem -noout -subject -issuer

# Melihat isi CSR
openssl req -in request.csr -text -noout

# Melihat detail private key
openssl rsa -in private.key -text -noout

# Cek apakah private key valid/tidak corrupt
openssl rsa -in private.key -check

# Melihat fingerprint sertifikat (SHA256)
openssl x509 -in cert.pem -noout -fingerprint -sha256
```

## 7. Verifikasi Sertifikat & Chain

```bash
# Verifikasi sertifikat terhadap CA bundle sistem
openssl verify cert.pem

# Verifikasi dengan CA chain kustom (misal intermediate + root)
openssl verify -CAfile chain.pem cert.pem

# Mengecek apakah private key cocok dengan sertifikat (modulus harus sama)
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in private.key | openssl md5

# Mengecek apakah CSR cocok dengan private key
openssl req -noout -modulus -in request.csr | openssl md5
```

## 8. Test Koneksi SSL/TLS (s_client & s_server)

Sangat berguna untuk debugging masalah koneksi HTTPS/TLS ke server.

```bash
# Test koneksi TLS ke server (tampilkan detail handshake & sertifikat)
openssl s_client -connect example.com:443

# Sama seperti di atas tapi dengan SNI (untuk server dengan banyak domain di 1 IP)
openssl s_client -connect example.com:443 -servername example.com

# Test dan langsung keluar setelah handshake (tanpa masuk mode interaktif)
echo | openssl s_client -connect example.com:443 -servername example.com

# Cek tanggal kedaluwarsa sertifikat server secara langsung
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null \
  | openssl x509 -noout -enddate

# Test versi TLS tertentu (misal paksa TLS 1.2)
openssl s_client -connect example.com:443 -tls1_2

# Membuat server TLS sederhana untuk testing (butuh cert.pem & private.key)
openssl s_server -accept 4433 -cert cert.pem -key private.key
```

## 9. Konversi Format Sertifikat (PEM, DER, PFX/P12)

Dibutuhkan saat sertifikat harus dipindah antar sistem (misal dari Linux ke Windows/IIS).

```bash
# PEM -> DER
openssl x509 -in cert.pem -outform der -out cert.der

# DER -> PEM
openssl x509 -in cert.der -inform der -out cert.pem -outform pem

# PEM (cert + key) -> PKCS#12 (.pfx/.p12), umum dipakai di Windows/Java
openssl pkcs12 -export -in cert.pem -inkey private.key -out cert.pfx \
  -certfile ca-chain.pem

# PKCS#12 (.pfx/.p12) -> PEM (extract semuanya)
openssl pkcs12 -in cert.pfx -out cert.pem -nodes

# Extract hanya private key dari file .pfx
openssl pkcs12 -in cert.pfx -nocerts -out private.key -nodes

# Extract hanya sertifikat dari file .pfx
openssl pkcs12 -in cert.pfx -clcerts -nokeys -out cert.pem
```

## 10. Hashing / Digest (MD5, SHA256, dll)

```bash
# Hash sebuah file dengan SHA256
openssl dgst -sha256 file.txt

# Hash dengan MD5
openssl dgst -md5 file.txt

# Hash string langsung (tanpa file)
echo -n "teks rahasia" | openssl dgst -sha256

# Menyimpan hasil hash ke file
openssl dgst -sha256 file.txt > file.sha256
```

## 11. Enkripsi & Dekripsi File Simetris

```bash
# Enkripsi file dengan AES-256-CBC (akan diminta password)
openssl enc -aes-256-cbc -salt -pbkdf2 -in rahasia.txt -out rahasia.enc

# Dekripsi file
openssl enc -aes-256-cbc -d -pbkdf2 -in rahasia.enc -out rahasia.txt

# Enkripsi dengan password langsung di command (kurang aman untuk history shell)
openssl enc -aes-256-cbc -salt -pbkdf2 -in rahasia.txt -out rahasia.enc -pass pass:passwordku

# Enkripsi menggunakan file password
openssl enc -aes-256-cbc -salt -pbkdf2 -in rahasia.txt -out rahasia.enc -pass file:pass.txt
```

## 12. Enkripsi & Dekripsi Asimetris (RSA)

Berbeda dengan enkripsi simetris, di sini dipakai sepasang kunci: **public key** untuk mengenkripsi dan **private key** untuk mendekripsi. Cocok untuk data kecil (secret key, password, token), bukan file besar, karena ukuran plaintext dibatasi oleh ukuran RSA key yang dipakai.

```bash
# Enkripsi menggunakan public key milik penerima
openssl pkeyutl -encrypt -pubin -inkey public.key -in pesan.txt -out pesan.enc

# Dekripsi menggunakan private key (hanya pemilik private key yang bisa membuka)
openssl pkeyutl -decrypt -inkey private.key -in pesan.enc -out pesan.txt

# Enkripsi dengan padding OAEP (lebih aman, direkomendasikan dibanding PKCS1 default)
openssl pkeyutl -encrypt -pubin -inkey public.key -in pesan.txt -out pesan.enc \
  -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256

openssl pkeyutl -decrypt -inkey private.key -in pesan.enc -out pesan.txt \
  -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256

# Cara lama (masih sering ditemui di tutorial, sudah deprecated tapi masih berfungsi)
openssl rsautl -encrypt -pubin -inkey public.key -in pesan.txt -out pesan.enc
openssl rsautl -decrypt -inkey private.key -in pesan.enc -out pesan.txt

# Tanda tangan digital (sign) menggunakan private key
openssl dgst -sha256 -sign private.key -out pesan.sig pesan.txt

# Verifikasi tanda tangan menggunakan public key milik pengirim
openssl dgst -sha256 -verify public.key -signature pesan.sig pesan.txt
```

> **Catatan:** Untuk mengenkripsi file besar, praktik yang benar adalah *hybrid encryption* — enkripsi file dengan AES (simetris, lihat bagian 11), lalu enkripsi **key AES**-nya saja dengan RSA (asimetris) seperti di atas.

## 13. Encoding Base64

```bash
# Encode file ke base64
openssl base64 -in file.bin -out file.b64

# Decode base64 kembali ke file asli
openssl base64 -d -in file.b64 -out file.bin

# Encode string langsung
echo -n "halo dunia" | openssl base64
```

## 14. Generate Angka Acak & Password

```bash
# Generate 16 byte random dalam format hex
openssl rand -hex 16

# Generate 32 byte random dalam format base64 (cocok untuk secret key/API key)
openssl rand -base64 32

# Generate angka random murni (raw bytes) dan simpan ke file
openssl rand -out random.bin 64
```

## 15. Membuat CA Sendiri & Menandatangani CSR

Berguna untuk membuat Certificate Authority internal (misal untuk jaringan kantor/lab).

```bash
# 1. Buat private key untuk Root CA
openssl genrsa -out ca.key 4096

# 2. Buat sertifikat Root CA self-signed (masa berlaku panjang, misal 10 tahun)
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.pem \
  -subj "/C=ID/O=Perusahaan Saya/CN=Root CA Internal"

# 3. Buat CSR untuk server yang akan ditandatangani oleh CA di atas
openssl req -new -key server.key -out server.csr -subj "/CN=server.internal"

# 4. Tandatangani CSR menggunakan Root CA sehingga jadi sertifikat resmi internal
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key -CAcreateserial \
  -out server.pem -days 365 -sha256 \
  -extfile <(printf "subjectAltName=DNS:server.internal")
```

## 16. HMAC

```bash
# Membuat HMAC-SHA256 dari sebuah file menggunakan secret key
openssl dgst -sha256 -hmac "kunci-rahasia" file.txt

# HMAC dari string langsung
echo -n "data penting" | openssl dgst -sha256 -hmac "kunci-rahasia"
```

## 17. Diffie-Hellman Parameters

Digunakan untuk konfigurasi Perfect Forward Secrecy di server web (Nginx/Apache).

```bash
# Generate parameter DH 2048-bit (bisa memakan waktu cukup lama)
openssl dhparam -out dhparam.pem 2048

# Versi lebih cepat untuk kebutuhan testing (kurang aman, jangan untuk produksi)
openssl dhparam -out dhparam.pem 1024
```

## 18. Certificate Revocation List (CRL)

Dipakai oleh CA untuk mencabut sertifikat yang sudah tidak valid.

```bash
# Melihat isi CRL
openssl crl -in ca.crl -text -noout

# Membuat CRL kosong dari CA (butuh index.txt & konfigurasi openssl.cnf)
openssl ca -config openssl.cnf -gencrl -out ca.crl

# Konversi CRL dari DER ke PEM
openssl crl -in ca.crl -inform der -out ca-crl.pem -outform pem
```

## 19. Benchmark Performa (speed)

```bash
# Benchmark kecepatan algoritma tertentu (misal AES-256-CBC)
openssl speed aes-256-cbc

# Benchmark RSA
openssl speed rsa2048

# Benchmark semua algoritma default sekaligus
openssl speed
```

## 20. Tips & Troubleshooting Umum

- **Cek kecocokan key & cert** dengan membandingkan hasil modulus (lihat bagian 7). Jika hash MD5 sama, berarti key cocok dengan sertifikatnya.
- **Selalu gunakan `-pbkdf2`** saat enkripsi dengan `openssl enc` di versi OpenSSL modern, karena default key derivation lama dianggap lemah.
- **Gunakan `-sha256` atau lebih tinggi**, hindari `-sha1`/`-md5` untuk keperluan keamanan (hanya aman dipakai untuk checksum non-keamanan).
- **`-nodes`** pada `openssl req`/`openssl x509` berarti "no DES", yaitu private key **tidak** dienkripsi dengan passphrase — praktis untuk server otomatis, tapi kurang aman jika file key bocor.
- **`pkeyutl` vs `rsautl`**: `rsautl` sudah deprecated sejak OpenSSL 3.0, gunakan `pkeyutl` untuk enkripsi/dekripsi asimetris pada implementasi baru.
- Jika muncul error `unable to load certificate` atau `unable to get local issuer certificate`, biasanya karena format file salah (PEM vs DER) atau chain sertifikat tidak lengkap — cek dengan perintah pada bagian 7.
- Simpan `.serial` dan `index.txt` dengan aman jika membuat CA sendiri (bagian 15), karena dibutuhkan setiap kali menandatangani CSR baru.
