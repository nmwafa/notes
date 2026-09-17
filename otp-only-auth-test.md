---
title: "OTP-Only Authentication Security Test Cases"
layout: default
---

# OTP-Only Authentication Security Test Cases

> Checklist pengujian keamanan untuk aplikasi web yang menggunakan **nomor HP + OTP** sebagai satu-satunya mekanisme autentikasi.

---

## 1. OTP Reuse

**Tujuan:** Memastikan OTP benar-benar hanya dapat digunakan sekali.

**Langkah uji:**

1. Request OTP untuk nomor HP akun uji.
2. Simpan OTP yang diterima.
3. Verify OTP dan pastikan login berhasil.
4. Kirim ulang request verifikasi menggunakan OTP yang sama.
5. Uji juga setelah logout atau session berakhir.

**Expected:**

- Penggunaan pertama berhasil.
- Penggunaan berikutnya ditolak sebagai OTP sudah digunakan/tidak valid.

**Indikasi temuan:**

- OTP yang sudah berhasil digunakan tetap valid.
- OTP lama dapat membuat session authenticated baru.

---

## 2. Resend OTP Invalidation

**Tujuan:** Memastikan OTP lama tidak tetap valid setelah meminta OTP baru.

**Langkah uji:**

1. Request OTP pertama (`OTP A`).
2. Gunakan fungsi **Resend OTP** sehingga menerima `OTP B`.
3. Coba verify menggunakan `OTP A`.
4. Coba verify menggunakan `OTP B`.

**Expected:**

- `OTP A` ditolak.
- `OTP B` menjadi OTP yang aktif.

**Indikasi temuan:**

- `OTP A` dan `OTP B` sama-sama dapat digunakan.

---

## 3. OTP Brute-Force Resistance

**Tujuan:** Menguji apakah server membatasi percobaan OTP.

**Dimensi pengujian:**

- Nomor HP sama + IP sama.
- Nomor HP sama + session berbeda.
- Nomor HP sama + User-Agent berbeda.
- Nomor HP sama + IP berbeda.
- Nomor HP berbeda + IP sama.

**Perhatikan:**

- HTTP status.
- Response body.
- Pesan error.
- Counter percobaan.
- Temporary lockout.
- CAPTCHA/challenge tambahan.
- Response timing.
- Apakah counter reset saat membuat session baru.

**Expected:**

- Jumlah percobaan dibatasi server-side.
- Membuat session baru tidak menghapus pembatasan yang semestinya.
- Tidak ada jalur sederhana untuk menghindari rate limit.

**Indikasi temuan:**

- OTP dapat dicoba terus tanpa pembatasan.
- Counter hanya diterapkan pada frontend.
- Rate limit dapat di-bypass dengan mengganti session/header/IP secara tidak semestinya.

---

## 4. Race Condition pada OTP Verification

**Tujuan:** Memastikan OTP tidak dapat dipakai secara bersamaan beberapa kali.

**Langkah uji:**

1. Request satu OTP.
2. Siapkan beberapa request verify dengan OTP yang sama.
3. Kirim request tersebut hampir bersamaan.
4. Bandingkan hasil masing-masing request.

**Expected:**

- Hanya satu request yang dapat sukses.
- Setelah satu sukses, OTP langsung dianggap terpakai.

**Indikasi temuan:**

```text
Request A -> SUCCESS
Request B -> SUCCESS
Request C -> SUCCESS
```

jika semuanya berhasil menggunakan OTP yang sama.

---

## 5. OTP Binding dengan Nomor HP

**Tujuan:** Memastikan OTP terikat pada nomor yang meminta OTP.

**Langkah uji:**

1. Request OTP untuk nomor A.
2. Catat OTP A.
3. Coba verifikasi menggunakan:
   - nomor A + OTP A
   - nomor B + OTP A
4. Ulangi dengan OTP dari nomor B.

**Expected:**

- OTP A hanya valid untuk nomor A.
- OTP B hanya valid untuk nomor B.

**Indikasi temuan:**

- OTP dapat dipindahkan ke nomor lain.
- Server memvalidasi OTP secara global tanpa binding identitas.

---

## 6. OTP Binding dengan Challenge / Verification ID

**Tujuan:** Memastikan OTP terikat pada challenge atau verification session yang benar.

**Langkah uji:**

Cari parameter seperti:

```text
challenge_id
verification_id
request_id
transaction_id
session_id
```

Kemudian buat dua proses:

```text
Challenge A -> OTP A
Challenge B -> OTP B
```

Uji kombinasi:

```text
Challenge A + OTP A
Challenge A + OTP B
Challenge B + OTP A
Challenge B + OTP B
```

**Expected:**

- Hanya pasangan challenge + OTP yang benar yang diterima.

**Indikasi temuan:**

- OTP valid terhadap challenge yang berbeda.

---

## 7. Authentication State Bypass

**Tujuan:** Menguji apakah endpoint authenticated dapat diakses tanpa melewati verifikasi OTP.

**Model state:**

```text
S0 = Belum meminta OTP
S1 = OTP sudah diminta
S2 = OTP berhasil diverifikasi
S3 = Authenticated
```

**Langkah uji:**

Coba mengakses endpoint S3 langsung dari:

```text
S0
S1
```

Contoh:

```text
GET /profile
GET /account
GET /orders
```

Tanpa menyelesaikan OTP.

**Expected:**

- Server menolak akses authenticated.

**Indikasi temuan:**

- Endpoint sensitif dapat diakses sebelum OTP berhasil.

---

## 8. Skip-Step / State Machine Bypass

**Tujuan:** Memastikan urutan autentikasi tidak dapat dilewati.

**Contoh flow normal:**

```text
/request-otp
        ↓
/verify-otp
        ↓
/login-complete
        ↓
/profile
```

**Langkah uji:**

Coba memanggil:

```text
/login-complete
```

tanpa `/verify-otp`.

Kemudian uji kombinasi state lain, misalnya:

```text
/request-otp -> /profile
/verify-otp  -> /login-complete
/login-complete -> /profile
```

**Expected:**

- Server menegakkan state authentication secara server-side.

**Indikasi temuan:**

- Step penting dapat dilewati.

---

## 9. Pre-Authentication Session Confusion

**Tujuan:** Menguji apakah session sebelum OTP dapat dipromosikan menjadi session authenticated secara tidak sah.

**Langkah uji:**

1. Buka halaman login.
2. Catat cookie/session sebelum OTP.
3. Request OTP.
4. Verify OTP.
5. Bandingkan token sebelum dan sesudah autentikasi.

**Perhatikan:**

```text
Set-Cookie
Authorization
session_id
access_token
refresh_token
```

**Expected:**

- Session authentication memiliki state yang benar.
- Session ID sensitif tidak dapat digunakan untuk melewati verifikasi.

---

## 10. Session Fixation

**Tujuan:** Memastikan session/token berubah setelah autentikasi.

**Langkah uji:**

1. Ambil session ID sebelum login.
2. Login dengan OTP valid.
3. Bandingkan session ID setelah login.

**Expected:**

```text
Session_before != Session_after
```

**Indikasi temuan:**

- Session yang sama dipromosikan begitu saja dari unauthenticated menjadi authenticated dan dapat digunakan secara tidak semestinya.

---

## 11. Session Invalidation setelah Logout

**Tujuan:** Memastikan token/session lama tidak tetap aktif setelah logout.

**Langkah uji:**

1. Login.
2. Simpan access token/session.
3. Logout.
4. Gunakan kembali token/session lama terhadap endpoint authenticated.

Contoh:

```http
GET /profile
Authorization: Bearer <OLD_TOKEN>
```

**Expected:**

```text
401 / 403
```

atau respons lain yang menunjukkan session sudah tidak valid.

**Indikasi temuan:**

- Token lama tetap dapat mengakses akun.

---

## 12. Multiple Sessions / Multiple Devices

**Tujuan:** Memahami perilaku session ketika satu akun login dari beberapa perangkat.

**Langkah uji:**

```text
Device A -> Login
Device B -> Login
```

Kemudian:

```text
Device A -> Logout
```

Uji apakah session Device B tetap aktif dan apakah perilakunya sesuai desain aplikasi.

**Perhatikan juga:**

- Apakah login baru mencabut session lama?
- Apakah semua session dapat dicabut?
- Apakah device management tersedia?
- Apakah token lama tetap hidup?

---

## 13. OTP Expiration

**Tujuan:** Memastikan TTL OTP benar-benar dipaksakan server-side.

**Langkah uji:**

1. Request OTP.
2. Catat waktu penerimaan.
3. Uji pada beberapa interval waktu.
4. Gunakan OTP yang sama setelah melewati masa berlaku yang seharusnya.

**Expected:**

- OTP kedaluwarsa setelah TTL.
- Validasi dilakukan di server, bukan hanya oleh UI.

**Indikasi temuan:**

- OTP tetap valid jauh setelah masa berlaku yang seharusnya.

---

## 14. Client-Side Timestamp Trust

**Tujuan:** Menguji apakah aplikasi mempercayai timestamp dari client.

**Cari parameter seperti:**

```text
timestamp
created_at
expires_at
client_time
issued_at
```

**Langkah uji:**

- Ubah nilai waktu yang dikirim client.
- Bandingkan hasil validasi.

**Expected:**

- Server menggunakan waktu server-side untuk menentukan TTL dan expiry.

**Indikasi temuan:**

- Client dapat memperpanjang masa validitas OTP dengan mengubah timestamp.

---

## 15. OTP Leakage di Response

**Tujuan:** Memastikan OTP tidak dikembalikan atau diekspos di sisi client.

**Periksa:**

```text
HTTP response
JSON response
HTML
JavaScript
console
localStorage
sessionStorage
cookies
debug information
error response
analytics payload
```

**Cari pola:**

```json
{
  "otp": "123456"
}
```

atau:

```json
{
  "code": "123456"
}
```

**Expected:**

- OTP hanya dikirim melalui channel OTP yang semestinya.
- Tidak ada OTP di response atau storage client yang tidak diperlukan.

---

## 16. OTP dalam URL

**Tujuan:** Menemukan OTP yang dikirim sebagai query parameter.

**Cari request seperti:**

```http
GET /verify?phone=62812...&otp=123456
```

**Periksa potensi paparan melalui:**

- Browser history.
- Proxy logs.
- Web server logs.
- Analytics.
- Referer.
- Monitoring systems.

**Expected:**

- Secret tidak diletakkan pada URL.
- HTTPS digunakan sepanjang flow autentikasi.

---

## 17. Manipulasi `verified` / `success` / Client State

**Tujuan:** Memastikan backend tidak mempercayai status autentikasi yang berasal dari client.

**Cari field seperti:**

```text
verified
otp_verified
isAuthenticated
success
authenticated
login_complete
```

**Langkah uji:**

- Ubah nilai boolean/status pada request.
- Amati apakah server memberikan session atau akses yang seharusnya membutuhkan OTP valid.

**Expected:**

- Backend menentukan status autentikasi berdasarkan hasil validasinya sendiri.

**Indikasi temuan:**

- Mengubah flag client dapat memicu autentikasi.

---

## 18. Parameter Pollution

**Tujuan:** Mencari perbedaan parsing parameter antara proxy/WAF/backend.

**Parameter yang relevan:**

```text
phone
otp
user_id
account_id
challenge_id
verification_id
device_id
```

**Contoh pola pengujian:**

```http
phone=A&phone=B
```

atau JSON dengan parameter duplikat sesuai format yang diterima aplikasi.

**Perhatikan:**

- Nilai mana yang dianggap server.
- Perbedaan parsing antar-layer.
- Perbedaan HTTP status/response.

**Expected:**

- Parameter duplikat ditolak atau diproses secara deterministik dan aman.

---

## 19. OTP Request Rate Limit

**Tujuan:** Menguji penyalahgunaan endpoint pengiriman OTP.

**Langkah uji:**

Kirim request berkali-kali pada akun uji.

Uji skenario:

```text
Nomor sama + IP sama
Nomor berbeda + IP sama
Nomor sama + session berbeda
```

**Perhatikan:**

- Jumlah SMS yang dapat dipicu.
- Cooldown.
- Daily/hourly limit.
- Perubahan response setelah limit tercapai.

**Potensi dampak:**

```text
OTP spam
SMS bombing
resource exhaustion
SMS provider cost abuse
```

---

## 20. Race Condition pada Request OTP

**Tujuan:** Menguji konsistensi challenge ketika banyak request OTP terjadi bersamaan.

**Langkah uji:**

1. Kirim beberapa `/request-otp` secara paralel.
2. Catat challenge/OTP yang dibuat.
3. Periksa OTP mana yang valid.
4. Pastikan OTP lama di-invalidasi sesuai desain.

**Expected:**

- State challenge konsisten.
- Sistem memiliki aturan jelas tentang OTP aktif.

**Indikasi temuan:**

- Banyak OTP berbeda semuanya tetap valid.
- Challenge saling tertukar.

---

## 21. Account Enumeration

**Tujuan:** Mengetahui apakah nomor HP yang terdaftar dapat dibedakan dari yang tidak terdaftar.

**Bandingkan untuk nomor uji yang diketahui valid dan invalid:**

```text
HTTP status
response body
response length
JSON fields
error message
response timing
headers
```

**Expected:**

- Respons sensitif tidak mengungkapkan apakah nomor tertentu terdaftar, atau perbedaan diminimalkan.

**Indikasi temuan:**

```text
"Phone number registered"
"Phone number not registered"
```

atau perbedaan teknis yang konsisten.

---

## 22. Phone Number Normalization

**Tujuan:** Memastikan satu nomor HP selalu direpresentasikan sebagai identitas yang sama.

**Contoh representasi untuk akun uji:**

```text
081234567890
6281234567890
+6281234567890
+62 812 3456 7890
62812-3456-7890
```

**Perhatikan:**

- Registration.
- OTP request.
- OTP verification.
- Login.
- Change phone.
- Account lookup.

**Expected:**

- Normalisasi dilakukan konsisten server-side.

**Indikasi temuan:**

- Satu nomor dapat menghasilkan lebih dari satu account identity.
- OTP request dan verification menggunakan identitas berbeda karena perbedaan format.

---

## 23. Duplicate Account / Registration Race

**Tujuan:** Menguji apakah nomor HP yang sama dapat menghasilkan account ganda.

**Langkah uji:**

- Jalankan proses registration terhadap nomor uji secara paralel.
- Gunakan variasi format nomor yang secara logis sama.

**Expected:**

- Nomor unik terhadap account sesuai desain.

**Indikasi temuan:**

- Terbentuk dua atau lebih account identity untuk nomor yang sama.

---

## 24. JWT / Access Token Analysis

**Tujuan:** Memeriksa token yang diterbitkan setelah OTP.

**Periksa jika aplikasi menggunakan JWT:**

```text
alg
typ
iss
aud
sub
exp
iat
jti
scope
role
user_id
phone
```

**Pengujian penting:**

- Apakah signature wajib valid?
- Apakah expiry ditegakkan?
- Apakah claim sensitif hanya digunakan sesuai otorisasi server?
- Apakah token memiliki entropy/keunikan yang memadai?

**Catatan:**

Jangan mengubah token pengguna lain. Gunakan token akun uji Anda sendiri untuk validasi.

---

## 25. Identity Parameter Manipulation

**Tujuan:** Memastikan identitas akun ditentukan dari server-side authentication context, bukan parameter client yang mudah diubah.

**Cari parameter:**

```text
user_id
account_id
customer_id
phone
profile_id
member_id
```

**Langkah uji aman:**

- Gunakan dua akun uji yang Anda miliki.
- Uji pertukaran identifier di endpoint yang memang Anda berhak akses.

**Expected:**

- Session A hanya dapat bertindak sebagai A.
- Mengubah identifier tidak mengubah identitas session secara tidak sah.

---

## 26. Change Phone Number Re-authentication

**Tujuan:** Menguji keamanan proses perubahan nomor HP.

Karena nomor HP merupakan authenticator utama, endpoint perubahan nomor harus mendapat perhatian khusus.

**Langkah uji:**

1. Login dengan akun uji menggunakan nomor A.
2. Buka fitur change phone.
3. Amati apakah aplikasi meminta autentikasi ulang.
4. Uji alur dengan nomor B.
5. Logout.
6. Coba login kembali menggunakan nomor lama dan baru.

**Perhatikan:**

- OTP nomor lama.
- OTP nomor baru.
- Re-authentication.
- Session rotation.
- Pengiriman notifikasi keamanan.

**Indikasi temuan:**

- Nomor authenticator dapat diganti tanpa kontrol autentikasi yang sesuai.

---

## 27. Re-authentication untuk Operasi Sensitif

**Tujuan:** Menguji apakah session lama dapat digunakan langsung untuk tindakan berisiko tinggi.

**Cari operasi seperti:**

```text
change phone
change email
delete account
withdraw
change bank account
add beneficiary
API key creation
security setting changes
```

**Langkah uji:**

- Login menggunakan akun uji.
- Biarkan session tetap aktif.
- Jalankan operasi sensitif tanpa OTP tambahan/re-authentication.
- Bandingkan dengan desain/requirement aplikasi.

**Expected:**

- Operasi sensitif mendapatkan perlindungan yang sesuai.

---

## 28. OTP Challenge Cross-Use

**Tujuan:** Memastikan challenge milik satu flow tidak dapat digunakan dalam flow lain.

**Skenario:**

```text
Login Challenge -> OTP A
Phone Change Challenge -> OTP B
```

Uji apakah:

```text
OTP A -> Phone Change
OTP B -> Login
```

dapat diterima.

**Expected:**

- Challenge/OTP hanya valid untuk context yang membuatnya.

---

## 29. Session Expiry setelah Login

**Tujuan:** Memastikan session authenticated memiliki expiration dan kontrol lifecycle yang sesuai.

**Perhatikan:**

```text
access token expiry
refresh token expiry
idle timeout
absolute timeout
logout
passwordless re-login
```

Untuk aplikasi yang sepenuhnya passwordless, pastikan expiry session tidak membuat OTP menjadi satu-satunya penghalang yang dapat diabaikan.

---

## 30. Refresh Token / Token Rotation

**Tujuan:** Menguji lifecycle refresh token jika digunakan.

**Langkah uji:**

1. Login menggunakan OTP.
2. Catat refresh token.
3. Gunakan refresh token.
4. Periksa apakah token baru diterbitkan.
5. Uji token lama setelah rotation.
6. Logout dan uji kembali.

**Expected:**

- Token lifecycle mengikuti desain keamanan aplikasi.
- Token yang telah dicabut tidak tetap valid tanpa alasan yang jelas.

---

# Test Matrix Utama

Gunakan matriks berikut untuk verifikasi OTP:

| Kondisi | Expected |
|---|---|
| Nomor benar + OTP benar | SUCCESS |
| Nomor benar + OTP salah | DENY |
| Nomor salah + OTP benar | DENY |
| OTP expired | DENY |
| OTP sudah digunakan | DENY |
| OTP lama setelah resend | DENY |
| Challenge salah + OTP benar | DENY |
| Session berbeda + OTP benar | DENY, sesuai binding desain |
| OTP tanpa request sebelumnya | DENY |
| Verify tanpa state yang benar | DENY |

---

# Session Test Matrix

| Kondisi | Expected |
|---|---|
| Akses endpoint sebelum OTP | DENY |
| Akses endpoint setelah OTP invalid | DENY |
| Token sebelum login dipakai sebagai authenticated token | DENY |
| Session setelah logout | DENY |
| Session expired | DENY |
| Manipulasi identifier | DENY |
| Login ulang | Session lifecycle konsisten |
| Change phone | Memerlukan kontrol autentikasi yang sesuai |

---

# Checklist Evidence

Untuk setiap temuan, dokumentasikan:

- [ ] Endpoint
- [ ] HTTP method
- [ ] Parameter
- [ ] Request sebelum modifikasi
- [ ] Request setelah modifikasi
- [ ] HTTP status
- [ ] Response body
- [ ] Response headers
- [ ] Cookie/session
- [ ] Timestamp
- [ ] Kondisi akun uji
- [ ] Expected behavior
- [ ] Actual behavior
- [ ] Dampak keamanan
- [ ] Langkah reproduksi minimal
- [ ] Screenshot/video bila diperlukan

---

# Prioritas Pengujian

## Critical / High Priority

- [ ] OTP brute-force / rate-limit bypass
- [ ] OTP reuse
- [ ] OTP dapat digunakan setelah resend
- [ ] OTP + phone mismatch
- [ ] OTP + challenge mismatch
- [ ] Race condition pada verify
- [ ] Authentication state bypass
- [ ] Skip-step / verification bypass
- [ ] Session fixation
- [ ] Logout tidak meng-invalidasi session
- [ ] Change phone tanpa autentikasi yang memadai
- [ ] Identity parameter manipulation
- [ ] Token/signature validation issue

## Medium Priority

- [ ] OTP expiry
- [ ] Account enumeration
- [ ] OTP leakage
- [ ] OTP di URL
- [ ] SMS rate-limit
- [ ] Request OTP race condition
- [ ] Phone number normalization
- [ ] Duplicate account race
- [ ] Refresh-token lifecycle

