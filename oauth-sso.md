# OAuth dan SSO

> credit: Deepseek

## Daftar Isi
- [Penjelasan Singkat OAuth dan SSO](#penjelasan-singkat-oauth-dan-sso)
  - [OAuth 2.0](#oauth-20)
  - [Single Sign-On (SSO)](#single-sign-on-sso)
- [Cara Kerja Singkat](#cara-kerja-singkat)
  - [Cara Kerja OAuth 2.0 (Authorization Code Grant)](#cara-kerja-oauth-20-authorization-code-grant)
  - [Cara Kerja SSO (menggunakan OIDC / SAML)](#cara-kerja-sso-menggunakan-oidc--saml)
- [Analisis Potensi Kerentanan dan Cara Eksploitasi](#analisis-potensi-kerentanan-dan-cara-eksploitasi)
  - [Kerentanan pada OAuth 2.0](#kerentanan-pada-oauth-20)
  - [Kerentanan pada SSO](#kerentanan-pada-sso)
- [Langkah Mitigasi](#langkah-mitigasi)
  - [Mitigasi OAuth 2.0](#mitigasi-oauth-20)
  - [Mitigasi SSO](#mitigasi-sso)
- [Checklist Pengujian Keamanan](#checklist-pengujian-keamanan)
  - [OAuth 2.0 / OIDC](#oauth-20--oidc)
  - [SSO (SAML)](#sso-saml)

---

## Penjelasan Singkat OAuth dan SSO

### OAuth 2.0
OAuth 2.0 adalah kerangka otorisasi yang memungkinkan aplikasi pihak ketiga memperoleh akses terbatas ke sumber daya pengguna di server lain tanpa harus membagikan kredensial pengguna. OAuth berfokus pada **otorisasi**, bukan autentikasi. Komponen utama:
- **Resource Owner** (pengguna)
- **Client** (aplikasi yang meminta akses)
- **Authorization Server** (server yang mengeluarkan token)
- **Resource Server** (server yang menyimpan data pengguna)

### Single Sign-On (SSO)
SSO adalah mekanisme autentikasi yang memungkinkan pengguna masuk ke beberapa aplikasi/sistem dengan satu set kredensial. Pengguna hanya perlu login sekali pada **Identity Provider (IdP)** dan kemudian dapat mengakses berbagai **Service Provider (SP)** tanpa login ulang. SSO sering dibangun di atas protokol seperti **OpenID Connect (OIDC)** (lapisan identitas di atas OAuth 2.0) atau **SAML 2.0**.

Perbedaan utama: OAuth 2.0 adalah tentang **otorisasi terdelegasi**, sedangkan SSO adalah tentang **autentikasi terpusat**.

---

## Cara Kerja Singkat

### Cara Kerja OAuth 2.0 (Authorization Code Grant)
1. Pengguna mengklik "Login dengan X" di aplikasi client.
2. Client mengarahkan browser pengguna ke Authorization Server dengan parameter seperti `client_id`, `redirect_uri`, `scope`, `state`.
3. Pengguna login dan menyetujui permintaan akses di Authorization Server.
4. Authorization Server mengembalikan **authorization code** ke browser pengguna yang kemudian dikirim ke `redirect_uri` client.
5. Client menukar authorization code dengan **access token** (dan opsional refresh token) melalui permintaan back-channel ke Authorization Server, disertai `client_secret`.
6. Client menggunakan access token untuk mengakses resource di Resource Server.

### Cara Kerja SSO (menggunakan OIDC / SAML)
**OIDC (OpenID Connect):**
- Mengalir di atas OAuth 2.0. Setelah authorization code ditukar, client mendapatkan **ID Token** (JWT) selain access token. ID Token berisi klaim identitas pengguna yang diverifikasi.
- Client dapat memvalidasi ID Token menggunakan kunci publik IdP, lalu membuat sesi lokal untuk pengguna.

**SAML 2.0:**
- Pengguna mengakses Service Provider (SP).
- SP mengarahkan pengguna ke Identity Provider (IdP) dengan SAML AuthnRequest.
- IdP mengautentikasi pengguna dan mengirimkan SAML Response (berisi Assertion) kembali ke SP melalui browser (POST atau redirect).
- SP memvalidasi tanda tangan digital assertion dan membuat sesi pengguna.

---

## Analisis Potensi Kerentanan dan Cara Eksploitasi

### Kerentanan pada OAuth 2.0

1. **Validasi `redirect_uri` yang Lemah / Tidak Ketat**
   - **Deskripsi:** Authorization Server menerima `redirect_uri` yang tidak persis sama dengan yang didaftarkan, misalnya subdomain liar, path traversal, atau parameter tambahan yang dapat dimanipulasi.
   - **Eksploitasi:** Penyerang mengelabui korban mengakses link OAuth dengan `redirect_uri` yang dikontrol penyerang. Setelah korban login, authorization code dikirim ke penyerang. Penyerang menukarnya dengan token dan mengambil alih akun korban.
   - **Cara menemukan:**
     - Ubah nilai `redirect_uri` ke domain penyerang atau domain mirip (misal `https://legitimate.com.attacker.com`).
     - Coba tambahkan path traversal (`/../malicious`) atau parameter terbuka di akhir (`https://app.com/callback?url=https://evil.com`).
     - Cek apakah Authorization Server melakukan pencocokan eksak atau hanya prefix/suffix.

2. **CSRF pada Aliran OAuth (State Parameter Missing/Weak)**
   - **Deskripsi:** Tidak ada atau lemahnya parameter `state` yang mengikat permintaan OAuth dengan sesi pengguna asli.
   - **Eksploitasi:** Penyerang memulai aliran OAuth menggunakan akunnya sendiri, mendapatkan authorization code, lalu menghentikan aliran. Penyerang memaksa korban untuk mengunjungi link callback yang berisi authorization code penyerang. Jika aplikasi client tidak memvalidasi `state`, sesi korban akan terikat dengan akun penyerang. Penyerang bisa melihat data sensitif korban yang disimpan di akun penyerang (account hijacking via forced linking).
   - **Cara menemukan:** Hapus parameter `state` dari request; jika masih berhasil, rentan. Buat dua sesi browser berbeda, tukar nilai `state`, lihat apakah client mendeteksi ketidakcocokan.

3. **Insufficient PKCE Protection (Authorization Code Injection)**
   - **Deskripsi:** Pada public clients (SPA, mobile), jika PKCE tidak diterapkan, authorization code dapat disadap dan digunakan oleh penyerang.
   - **Eksploitasi:** Penyerang menangkap authorization code dari korban melalui malicious app, log, atau referer header. Karena tidak ada `code_verifier`, penyerang dapat langsung menukar code dengan token.
   - **Cara menemukan:** Periksa apakah aliran Authorization Code memerlukan `code_challenge`/`code_verifier`. Coba tukar code tanpa mengirimkan `code_verifier`.

4. **Kebocoran Token melalui URL Fragment, Referer, atau Log**
   - **Deskripsi:** Token yang dikembalikan di URL fragment (Implicit grant) atau URL query string dapat terekspos di log server, riwayat browser, atau header Referer.
   - **Eksploitasi:** Jika aplikasi menggunakan Implicit Flow, token di fragment bisa disadap oleh skrip jahat melalui `Referer` ke situs eksternal. Cukup cek apakah token muncul di request menuju third-party.
   - **Cara menemukan:** Pantau lalu lintas jaringan, lihat apakah access token dikirim ke domain pihak ketiga melalui Referer. Gunakan skrip yang mengakses `window.location.hash`.

5. **Client Secret pada Public Clients**
   - **Deskripsi:** Aplikasi client-side (SPA, mobile) tidak dapat menyimpan `client_secret` dengan aman, namun masih digunakan.
   - **Eksploitasi:** Penyerang mengekstrak `client_secret` dari kode aplikasi, lalu menyamar sebagai aplikasi resmi untuk mendapatkan token dengan hak akses lebih.
   - **Cara menemukan:** Dekompilasi APK/IPA, cari di source JavaScript, atau strings.

6. **Token Scope yang Tidak Dibatasi / Privilege Escalation**
   - **Deskripsi:** Client meminta scope yang terlalu luas, atau server tidak memvalidasi scope yang diminta.
   - **Eksploitasi:** Ubah parameter `scope` pada request OAuth untuk mencakup akses ke resource sensitif. Jika server mengeluarkan token dengan scope tersebut, penyerang dapat mengakses data lain.
   - **Cara menemukan:** Modifikasi nilai `scope` (tambah/ubah) dan periksa apakah token yang dihasilkan memberikan akses lebih.

### Kerentanan pada SSO

1. **SAML Signature Bypass / XML Signature Wrapping (XSW)**
   - **Deskripsi:** Implementasi validasi SAML assertion hanya memeriksa elemen yang salah akibat manipulasi struktur XML, sehingga penyerang bisa menyisipkan assertion palsu yang lolos verifikasi.
   - **Eksploitasi:** Penyerang mencegat SAML Response, mengubah struktur XML (memindahkan signature ke elemen yang tidak divalidasi), dan memasukkan elemen `Assertion` buatan dengan identitas korban. Jika SP hanya mengambil assertion pertama tanpa memvalidasi keseluruhan pohon, penyerang bisa masuk sebagai korban.
   - **Cara menemukan:** Kirim SAML Response yang telah dimodifikasi dengan berbagai teknik XSW. Gunakan tools seperti `SAML Raider`.

2. **SAML Replay Attack**
   - **Deskripsi:** SAML Response tidak memiliki batasan waktu atau mekanisme anti-replay yang kuat, sehingga response yang sah bisa digunakan kembali.
   - **Eksploitasi:** Penyerang mendapatkan SAML Response yang masih valid, lalu mengirimkannya kembali ke SP untuk mendapatkan sesi.
   - **Cara menemukan:** Simpan SAML Response yang dikeluarkan untuk sesi Anda, logout, lalu replay response tersebut. Jika berhasil, tidak ada pemeriksaan `NotOnOrAfter` atau `ID` unik yang dicegah pengulangan.

3. **Penerbitan Token untuk Identity Provider yang Tidak Terpercaya (IdP Confusion)**
   - **Deskripsi:** SP tidak membatasi IdP mana yang dapat menerbitkan assertion. Penyerang bisa mendirikan IdP sendiri dan mengarahkan korban untuk login di sana, lalu assertion dari IdP jahat diterima oleh SP.
   - **Eksploitasi:** Jika SP memungkinkan pengguna memilih IdP, penyerang mengirim link SSO yang mengarahkan ke IdP penyerang. SP memvalidasi assertion dari IdP itu dan mengizinkan akses.
   - **Cara menemukan:** Coba daftarkan IdP Anda sendiri di SP, atau modifikasi parameter `Issuer`/`entityID` pada SAML AuthnRequest menuju IdP arbitrer.

4. **Session Fixation pada SSO**
   - **Deskripsi:** SP menerima ID sesi dari luar sebelum autentikasi selesai, dan setelah SSO berhasil sesi itu tidak diregenerasi.
   - **Eksploitasi:** Penyerang menetapkan ID sesi tertentu pada korban (misal lewat URL), korban login via SSO, sesi tetap sama, penyerang menggunakan ID sesi tersebut untuk mengakses akun korban.
   - **Cara menemukan:** Sebelum login, catat cookie sesi. Lakukan login SSO, periksa apakah cookie sesi berubah. Jika tidak, mungkin rentan session fixation.

5. **Kebocoran Klaim / Informasi Sensitif pada ID Token / SAML Assertion**
   - **Deskripsi:** ID Token atau assertion berisi informasi pribadi yang tidak seharusnya dibagikan ke client, atau client meneruskannya tanpa perlu.
   - **Eksploitasi:** Tidak langsung, tapi penyerang bisa mencuri token dan membaca informasi pengguna (PII disclosure).
   - **Cara menemukan:** Decode ID Token (JWT) atau bongkar SAML assertion, periksa atribut yang dikirim. Terlalu banyak informasi.

6. **Tidak Validasi Audience / Recipient**
   - **Deskripsi:** OIDC/SAML tidak memeriksa bahwa token/assertion ditujukan untuk client/SP ini, sehingga token yang dikeluarkan untuk aplikasi lain bisa digunakan.
   - **Eksploitasi:** Dapatkan ID Token yang valid untuk aplikasi A, gunakan sebagai kredensial di aplikasi B yang memiliki IdP sama. Jika aplikasi B tidak memvalidasi `aud` (audience) dengan benar, token diterima.
   - **Cara menemukan:** Ambil token/assertion dari aplikasi lain yang menggunakan IdP sama, kirim ke endpoint aplikasi target.

---

## Langkah Mitigasi

### Mitigasi OAuth 2.0
- **Validasi ketat `redirect_uri`:** Hanya izinkan URI yang terdaftar eksak, tidak menerima wildcard atau substring. Gunakan whitelist.
- **Wajibkan parameter `state`:** Bangkitkan nilai `state` acak per request, ikat dengan sesi pengguna, verifikasi saat callback.
- **Gunakan PKCE (Proof Key for Code Exchange):** Semua aliran Authorization Code wajib menggunakan PKCE, terutama untuk public clients. Server harus menolak request tanpa `code_challenge`.
- **Hindari Implicit Flow:** Beralih ke Authorization Code dengan PKCE untuk SPA. Jangan kirim token di URL fragment.
- **Jangan embed `client_secret` di client publik:** Gunakan alur tanpa secret atau gunakan backend-for-frontend pattern.
- **Batasi scope:** Hanya berikan scope minimum yang diperlukan. Validasi scope yang diminta, abaikan scope yang tidak diizinkan.
- **Gunakan state dan nonce:** Nonce di ID token untuk mencegah replay.
- **Simpan token dengan aman:** Access/refresh token di httpOnly secure cookie (backend) atau penyimpanan aman platform (mobile).

### Mitigasi SSO
- **Validasi tanda tangan digital dengan benar:** Gunakan perpustakaan yang tahan terhadap XML Signature Wrapping. Verifikasi keseluruhan pohon XML, bukan hanya elemen tertentu. Selalu gunakan `WantAssertionsSigned` true.
- **Anti-replay:** Periksa `NotOnOrAfter` / `NotBefore` pada assertion. Catat ID assertion (`InResponseTo`, `ID`) dan tolak yang sudah pernah diproses dalam jendela waktu tertentu.
- **Batasi IdP yang diterima:** Hanya percayai assertion dari IdP yang sudah dikonfigurasi secara eksplisit (verifikasi `Issuer` dan entityID).
- **Regenerasi sesi setelah login:** Selalu buat ID sesi baru setelah sukses SSO, hapus sesi sebelumnya.
- **Validasi audience/recipient:** Periksa `aud` claim di ID Token atau `Audience`/`Recipient` di SAML. Pastikan sesuai dengan identitas aplikasi.
- **Minimalkan atribut yang dilepaskan:** IdP hanya mengirim atribut yang diperlukan. SP tidak boleh meminta data berlebih.
- **Gunakan HTTPS dan binding yang aman:** Untuk SAML, gunakan POST binding, hindari redirect binding untuk response sensitif. Lindungi dari MITM.
- **Gunakan short-lived token/assertion:** Masa berlaku pendek, dan implementasikan single-use assertion.

---

## Checklist Pengujian Keamanan

### OAuth 2.0 / OIDC
- **`redirect_uri` divalidasi secara ketat**
  - Pencocokan persis (bukan prefix/regex)
  - Tidak ada open redirect
  - Tidak ada wildcard
- **Parameter `state` ada dan divalidasi**
  - Acak, tidak dapat diprediksi
  - Terikat pada sesi
  - Sekali pakai
- **Kode otorisasi (`authorization code`)**
  - Hanya dapat digunakan sekali
  - Masa berlaku pendek (5–10 menit)
  - Terikat pada `client_id`
- **Keamanan token**
  - Masa berlaku pendek
  - Scope diberikan secara tepat
  - Disimpan dengan aman (tidak di localStorage, tidak di URL)
  - Tidak muncul di URL (query/fragment)
- **PKCE diterapkan (untuk public clients)**
  - `code_verifier` divalidasi dengan benar
  - Gunakan metode `S256` (bukan plain)
- **Autentikasi client**
  - `client_secret` terlindungi dengan benar
  - Tidak terekspos di kode sisi klien (JS, APK, IPA)
- **Penautan akun (account linking)**
  - Verifikasi email sebelum menautkan
  - Tidak ada penggabungan akun otomatis tanpa konfirmasi
- **Validasi token**
  - `iss` (issuer) diperiksa
  - `aud` (audience) diperiksa
  - `exp` (expiration) diperiksa
  - Tanda tangan diverifikasi
- **Keamanan refresh token**
  - Rotasi setiap kali digunakan (refresh token rotation)
  - Pencabutan (revocation) didukung
  - Masa hidup terbatas

### SSO (SAML)
- Apakah SAML Response/Assertion divalidasi tanda tangannya dengan benar (cek XSW – coba manipulasi struktur XML)?
- Apakah `NotOnOrAfter` dan `NotBefore` di assertion diperiksa ketat?
- Apakah ada perlindungan anti-replay (pengecekan ID assertion, `InResponseTo`)?
- Apakah SP menolak assertion dari IdP yang tidak dikenal? (Uji dengan IdP palsu atau issuer berbeda)
- Apakah sesi baru dibuat setelah login SSO? (Uji session fixation: set cookie lama, lalu login; cookie harus berubah)
- Apakah ID Token / assertion memvalidasi `aud` (audience) sesuai dengan client/SP?
- Apakah atribut yang dikirimkan IdP dibatasi hanya yang diperlukan? (Cek data sensitif di assertion/token)
- Apakah binding SAML yang digunakan aman (POST) dan tidak rentan terhadap MITM?
- Apakah Single Logout (SLO) diimplementasikan dengan benar untuk menghapus sesi di semua SP?
- Apakah error handling tidak mengungkap informasi sensitif (stack trace, assertion mentah)?
- Apakah URL callback/reply dikonfigurasi dengan ketat (untuk SAML) dan tidak bisa dimanipulasi?
- Apakah parameter `RelayState` divalidasi dan tidak membuka redirect terbuka?
- Apakah endpoint SSO dilindungi CSRF pada alur initiated by SP? (state/nonce)

---

> Seluruh checklist di atas dapat digunakan sebagai dasar pengujian penetrasi atau audit keamanan. Pastikan setiap poin diuji secara manual menggunakan burp suite, SAML Raider, atau tools sejenis, serta disesuaikan dengan konteks aplikasi.
