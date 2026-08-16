---
title: "Prototype Pollution"
layout: default
---

# Prototype Pollution

## 1. Penjelasan Singkat

### Apa itu Prototype Pollution?
Prototype Pollution adalah kerentanan yang khas pada JavaScript, di mana penyerang berhasil menyuntikkan properti ke dalam **prototype** sebuah objek (paling sering `Object.prototype`). Karena hampir semua objek di JavaScript mewarisi properti dari `Object.prototype` melalui *prototype chain*, properti yang berhasil "dikotori" (polluted) tersebut akan otomatis muncul di **semua objek** dalam aplikasi — termasuk objek yang dibuat setelah polusi terjadi.

Secara teknis, setiap kali sebuah properti diakses pada suatu objek, JavaScript engine akan:
1. Mengecek apakah properti tersebut ada langsung pada objek itu.
2. Jika tidak ada, engine akan menelusuri rantai prototype-nya secara rekursif sampai ditemukan atau sampai mencapai `null`.

Celah ini biasanya muncul lewat properti "ajaib" seperti `__proto__`, `constructor.prototype`, atau `prototype`, terutama pada kode yang melakukan **merge/clone/set** objek secara rekursif dari input yang tidak tepercaya (misalnya body JSON dari request HTTP) tanpa memfilter key tersebut.

### Sumber, Sink, dan Gadget
Tiga komponen kunci untuk membangun eksploitasi:
- **Source** — titik masuk (input) yang memungkinkan penyerang menambahkan properti sembarangan ke prototype (query string, JSON body, hash URL, `postMessage`, dsb).
- **Gadget** — properti yang nantinya dibaca oleh aplikasi tanpa validasi, sehingga nilai hasil polusi ikut terpakai dalam logika aplikasi.
- **Sink** — fungsi atau elemen DOM yang mengeksekusi/memproses nilai gadget tersebut (misalnya `eval`, `innerHTML`, pembuatan child process, pengecekan hak akses, dsb) sehingga menghasilkan dampak nyata seperti XSS atau RCE.

### Client-side vs Server-side

| Aspek | Client-side | Server-side |
|---|---|---|
| Lokasi eksekusi | Browser korban | Server aplikasi (umumnya Node.js) |
| Dampak umum | DOM XSS, bypass logic UI | DoS, tampering logika bisnis, privilege escalation, hingga Remote Code Execution |
| Deteksi | Relatif mudah lewat console/DevTools & DOM Invader | Sulit dideteksi secara black-box karena efeknya tidak selalu terlihat dan mudah memicu crash/DoS |

### Contoh Pola Kode Rentan (ilustrasi umum)
Pola klasik yang rentan adalah fungsi *deep merge* atau *set nested property* yang tidak memfilter key berbahaya:

```javascript
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === 'object') {
      if (!target[key]) target[key] = {};
      merge(target[key], source[key]); // tidak memfilter __proto__/constructor
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// Input JSON dari user: {"__proto__": {"isAdmin": true}}
merge({}, JSON.parse(userInput));
// Setelah ini, SEMUA objek baru mewarisi properti isAdmin = true
```

---

## 2. Panduan Pengujian

### A. Persiapan
1. Tentukan lingkup (scope) pengujian — endpoint mana yang menerima input terstruktur (JSON, query string, form).
2. Siapkan proxy intersepsi seperti Burp Suite atau OWASP ZAP untuk memodifikasi request.
3. Pastikan memiliki izin/otorisasi eksplisit sebelum menguji, khususnya untuk teknik yang bisa memicu crash server.

### B. Pengujian Client-Side (DOM-based)
1. Identifikasi *source* — cari titik input yang datanya dipetakan ke objek JavaScript: parameter URL, fragment/hash, atau `postMessage`.
2. Coba suntikkan payload dasar melalui parameter, misalnya:
   - `?__proto__[test]=polluted`
   - `?__proto__.test=polluted`
   - `?constructor[prototype][test]=polluted`
3. Jika menggunakan Burp Suite, aktifkan fitur Prototype Pollution pada DOM Invader untuk otomatis mendeteksi source dari URL maupun objek JSON yang dikirim lewat web message.
4. Verifikasi polusi berhasil dengan membuka console browser dan mengecek:
   ```javascript
   console.log(Object.prototype.test); // seharusnya "polluted" jika berhasil
   ```
5. DOM Invader juga dapat memindai gadget yang tersedia, lalu otomatis membuat proof-of-concept exploit yang menggabungkan source, gadget, dan sink untuk mengonfirmasi XSS.
6. Jika ditemukan gadget yang mengarah ke sink berbahaya (`eval`, `innerHTML`, `document.write`, dsb), coba bangun payload untuk memicu `alert()` sebagai bukti konsep XSS non-destruktif.

### C. Pengujian Server-Side (Black-box, Node.js/Express dsb.)
1. Perlu diperhatikan bahwa mempolusi objek di lingkungan server-side dengan properti nyata sering kali justru merusak fungsionalitas aplikasi atau bahkan membuat server down — inilah tantangan utama pengujian black-box untuk kerentanan ini.
2. Gunakan teknik non-destruktif hasil riset PortSwigger, misalnya memanipulasi properti status kesalahan HTTP:
   - Framework server-side JavaScript seperti Express memungkinkan pengembang mengatur status HTTP kustom; saat terjadi error, server bisa mengembalikan objek error dalam format JSON di body respons sebagai detail tambahan.
   - Coba kirim body JSON yang mempolusi `__proto__.status` atau `__proto__.headersSent` dengan nilai unik, lalu amati apakah perilaku/response status berubah secara tidak wajar.
3. Salah satu contoh teknik: mempolusi properti `status` pada `Object.prototype` lalu memeriksa apakah kode status error yang biasanya konsisten (mis. 510) berubah — jika berubah, kemungkinan besar prototype berhasil dipolusi.
4. Kirim payload melalui berbagai lokasi: body JSON, parameter query dalam format JSON, maupun `application/x-www-form-urlencoded` yang di-parse jadi objek bertingkat.
5. Jika memiliki akses source code (uji whitebox):
   - Untuk aplikasi Node.js, jalankan dengan flag `--inspect` atau `--inspect-brk` sehingga bisa terhubung ke Chrome DevTools lewat `chrome://inspect` untuk melakukan debugging langsung terhadap proses polusi prototype.
   - Telusuri fungsi merge/extend/clone/set kustom atau dependensi pihak ketiga yang diketahui rentan (lihat daftar CVE di atas), lalu cek apakah key `__proto__`, `constructor`, atau `prototype` difilter.
6. Setelah polusi berhasil dikonfirmasi secara non-destruktif, baru evaluasi dampak lanjutan (mis. bypass autentikasi/otorisasi) di lingkungan yang aman/terisolasi.

### D. Tools yang Direkomendasikan
| Tool | Fungsi | Catatan |
|---|---|---|
| Burp Suite DOM Invader | Deteksi & PoC otomatis untuk client-side prototype pollution | Fitur ini nonaktif secara default agar tidak mengganggu fungsi situs target; aktifkan lewat menu setting DOM Invader. |
| Burp Server-Side Prototype Pollution Scanner (BApp) | Memindai request yang mengandung body JSON untuk mendeteksi kerentanan server-side, dengan opsi param scan maupun full scan. | Ekstensi resmi PortSwigger, hasil bisa dilihat di Burp Pro/Community |
| ppfuzz | Tool berbasis Rust untuk memindai kerentanan client-side prototype pollution. | Butuh Chrome/Chromium terpasang |
| ppmap | Scanner/exploitation tool berbasis Go yang mengeksploitasi gadget XSS client-side yang sudah dikenal secara otomatis. | Cocok untuk validasi cepat gadget yang umum |
| proto-find, PPScan | Tool tambahan serta ekstensi browser untuk otomatis memindai halaman yang dikunjungi terhadap kerentanan prototype pollution. | Alternatif ringan untuk eksplorasi awal |

---

## 3. Checklist Pengujian

### Persiapan & Ruang Lingkup
- [ ] Sudah mendapat izin/otorisasi tertulis untuk menguji target (bukan sistem produksi orang lain tanpa izin)
- [ ] Ruang lingkup endpoint yang menerima input terstruktur (JSON/query/form) sudah dipetakan
- [ ] Proxy intersepsi (Burp/ZAP) sudah disiapkan dan dikonfigurasi

### Client-Side
- [ ] Semua parameter URL, fragment, dan pesan `postMessage` sudah diperiksa sebagai potensi source
- [ ] Payload dasar `__proto__[x]=y`, `__proto__.x=y`, dan `constructor[prototype][x]=y` sudah dicoba
- [ ] DOM Invader (atau tool sejenis) sudah diaktifkan dan dijalankan pada halaman target
- [ ] Polusi properti sudah diverifikasi via console (`Object.prototype.<key>`)
- [ ] Gadget yang mengarah ke sink berbahaya (eval, innerHTML, dsb) sudah diidentifikasi
- [ ] PoC XSS non-destruktif (mis. `alert()`) sudah divalidasi bila gadget ditemukan

### Server-Side
- [ ] Endpoint yang melakukan merge/extend/clone/set objek dari input user sudah diidentifikasi
- [ ] Teknik non-destruktif (mis. manipulasi properti status error) sudah dicoba sebelum teknik yang berisiko DoS
- [ ] Body JSON, query JSON, dan form-urlencoded bertingkat sudah diuji sebagai vektor
- [ ] Jika ada source code: fungsi merge kustom & dependensi pihak ketiga (lodash, minimist, dsb.) sudah ditelusuri versi dan riwayat CVE-nya
- [ ] Debugging via `--inspect`/`--inspect-brk` sudah dilakukan (jika whitebox & tersedia)
- [ ] Efek lanjutan (bypass autentikasi/otorisasi, DoS, RCE) hanya diuji di lingkungan aman/terisolasi
- [ ] Tidak menjalankan teknik yang berpotensi merusak/mematikan layanan pada sistem yang sedang digunakan pengguna lain

### Dependensi & Kode
- [ ] Semua dependensi npm sudah dicek terhadap advisory prototype pollution (`npm audit`, Snyk, GitHub Advisory Database)
- [ ] Fungsi merge/set kustom memfilter key `__proto__`, `constructor`, dan `prototype`
- [ ] Objek yang menampung data user dibuat dengan `Object.create(null)` atau menggunakan `Map`/`Set` bila memungkinkan
- [ ] Validasi skema (mis. JSON Schema) diterapkan pada input yang akan dipetakan ke objek

### Pelaporan
- [ ] Source, gadget, dan sink yang terlibat sudah didokumentasikan dengan jelas
- [ ] Dampak (XSS, DoS, privilege escalation, RCE) sudah dinilai sesuai konteks aplikasi
- [ ] Langkah reproduksi non-destruktif sudah dicatat agar tim developer bisa memverifikasi ulang
- [ ] Rekomendasi mitigasi sudah disertakan dalam laporan

---

## 4. Ringkasan Mitigasi (untuk tim development)
- Gunakan `new Set()` atau `new Map()` sebagai pengganti object literal ketika memungkinkan, karena struktur data ini tidak mewarisi dari `Object.prototype` dengan cara yang sama.
- Jika objek tetap harus digunakan, buat dengan `Object.create(null)` agar tidak mewarisi dari `Object.prototype`.
- `Object.freeze()` dan `Object.seal()` dapat digunakan untuk mencegah modifikasi prototype bawaan, meskipun berpotensi merusak aplikasi jika ada pustaka yang memang bergantung pada modifikasi prototype tersebut.
- Validasi/sanitasi setiap key input sebelum dipakai dalam operasi merge/set/clone rekursif — tolak key `__proto__`, `constructor`, dan `prototype`.
- Rutin memperbarui dependensi dan memantau advisory keamanan (npm audit / Snyk / GitHub Advisories).
