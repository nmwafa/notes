# Check List Mitigasi Ransomware

>Sumber: [eucyberacademy.com](https://drive.google.com/file/d/1gUqLR4zCjOT_S0eBXQDH1z0xMEGuJaby/view?usp=sharing)

---

Daftar isi:

1. [Manajemen Akses](#manajemen-akses)
2. [Edukasi Keamanan Siber](#edukasi-keamanan-siber)
3. [Manajemen Data](#manajemen-data)
4. [Kebijakan dan Manajemen Konfigurasi](#kebijakan-dan-manajemen-konfigurasi)
5. [Pemeliharaan dan Perbaikan](#pemeliharaan-dan-perbaikan)
6. [Solusi Keamanan Teknis](#solusi-keamanan-teknis)
7. [Deteksi Aktivitas Anomali dan Pemahaman Dampak](#deteksi-aktivitas-anomali-dan-pemahaman-dampak)
8. [Eksekusi Respon terhadap Insiden Terdeteksi](#eksekusi-respon-terhadap-insiden-terdeteksi)
9. [Analisis Efektif untuk Respon dan Pemulihan](#analisis-efektif-untuk-respon-dan-pemulihan)
10. [Mitigasi Penahanan Peristiwa dan Resolusi](#mitigasi-penahanan-peristiwa-dan-resolusi)
11. [Peningkatan Respon Melalui Pembelajaran](#peningkatan-respon-melalui-pembelajaran)
12. [Eksekusi dan Pemeliharaan Proses Pemulihan](#eksekusi-dan-pemeliharaan-proses-pemulihan)
13. [Koordinasi Pemulihan dengan Pihak Internal dan Eksternal](#koordinasi-pemulihan-dengan-pihak-internal-dan-eksternal)

---

## Manajemen Akses
- **Kelola Identitas dan Kredensial:** Manajemen kredensial yang tepat sangat penting karena ransomware sering dimulai dengan kebocoran kredensial.
- **Kelola Akses Jarak Jauh:** Mengelola akses jarak jauh membantu menjaga integritas sistem dan menggunakan otentikasi multi-faktor mengurangi risiko kebocoran akun.
- **Kelola Izin Akses dan Otorisasi:** Menerapkan prinsip hak istimewa paling sedikit dan pemisahan tugas mencegah akses yang tidak perlu dan mengurangi risiko.
- **Lindungi Integritas Jaringan:** Segmentasi jaringan membatasi penyebaran ransomware, penting untuk memisahkan jaringan IT dan OT.

## Edukasi Keamanan Siber
- **Menginformasikan dan Melatih Pengguna:** Mengedukasi pengguna mengurangi risiko praktik tidak aman yang menyebabkan serangan ransomware.

## Manajemen Data
- **Mempertahankan Kapasitas yang Memadai:** Memastikan ketersediaan, termasuk cadangan (backup) offline, meminimalkan dampak ransomware.
- **Menerapkan Perlindungan terhadap Kebocoran Data:** Mencegah kebocoran data adalah kunci karena pemerasan ganda (_double extortion_) umum terjadi dalam serangan ransomware.
- **Menggunakan Mekanisme Pemeriksaan Integritas:** Mendeteksi pembaruan perangkat lunak yang telah dirusak dapat mencegah penyisipan malware.
- **Memisahkan Lingkungan Pengembangan dari Lingkungan Produksi:** Segregasi ini menghentikan penyebaran ransomware ke dalam sistem produksi.

## Kebijakan dan Manajemen Konfigurasi
- **Membuat dan Memelihara Konfigurasi Dasar (_Baseline_):** Konfigurasi dasar mendeteksi perubahan yang tidak sah, yang mengindikasikan kemungkinan adanya serangan.
- **Menerapkan Proses Kontrol Perubahan Konfigurasi:** Proses yang tepat menegakkan pembaruan keamanan dan mencegah pengenalan malware.
- **Melakukan, Memelihara, dan Menguji Cadangan (_Backup_):** Cadangan yang teratur dan aman sangat penting untuk pemulihan dari ransomware.
- **Menerapkan dan Mengelola Rencana Respons dan Pemulihan:** Rencana yang komprehensif dan teruji memastikan respons yang tepat terhadap ancaman ransomware.

## Pemeliharaan dan Perbaikan
- **Menyetujui, Mencatat, dan Melakukan Pemeliharaan Jarak Jauh:** Pemeliharaan jarak jauh yang terkelola mencegah akses tidak sah yang dapat memperkenalkan malware.

## Solusi Keamanan Teknis
- **Mengelola Catatan Audit/Log:** Catatan audit membantu dalam mendeteksi perilaku yang tidak diharapkan, mendukung respons dan pemulihan.
- **Memasukkan Prinsip Fungsionalitas Paling Rendah (_Least Functionality_):** Prinsip ini dapat mencegah pergerakan di antara sistem target potensial, sehingga mengurangi risiko penyebaran.

## Deteksi Aktivitas Anomali dan Pemahaman Dampak
- **Pengumpulan dan Korelasi Data Peristiwa (_Event_):** Memanfaatkan berbagai sumber dan sensor dengan solusi SIEM meningkatkan visibilitas jaringan, deteksi dini ransomware, dan pemahaman tentang perambatannya melalui jaringan.
- **Penentuan Dampak:** Memahami dampak peristiwa menginformasikan prioritas respons dan pemulihan selama serangan ransomware.
- **Pemantauan Sistem Informasi dan Aset:** Pemantauan jaringan, aktivitas personel, dan kode berbahaya untuk mendeteksi potensi peristiwa; personel, koneksi, perangkat, dan perangkat lunak yang tidak sah; serta pemindaian kerentanan. Hal ini membantu dalam deteksi dini dan pencegahan.
- **Proses dan Prosedur Deteksi:** Memastikan kesadaran akan peristiwa anomali, akuntabilitas, kepatuhan, peningkatan berkelanjutan, dan komunikasi tepat waktu mengenai informasi deteksi.

## Eksekusi Respon terhadap Insiden Terdeteksi
- **Eksekusi Rencana Respons:** Eksekusi segera membantu menghentikan kerusakan, penyebaran infeksi, dan meminimalkan kerusakan lebih lanjut termasuk bahaya reputasi atau hukum.
- **Kegiatan Terkoordinasi dengan Pemangku Kepentingan:** Memastikan personel mengetahui peran mereka, insiden dilaporkan secara konsisten, dan pertukaran informasi terjadi dengan pemangku kepentingan internal dan eksternal untuk kesadaran keamanan siber yang lebih luas.

## Analisis Efektif untuk Respon dan Pemulihan
- **Investigasi dan Pemahaman Dampak:** Investigasi segera terhadap pemberitahuan dan pemahaman dampak insiden membantu dalam penahanan (containment) dini dan prioritas yang tepat selama pemulihan.
- **Forensik dan Analisis Kerentanan:** Kegiatan forensik dan analisis kerentanan membantu dalam penahanan, pembasmian, pencegahan serangan di masa depan, dan pemulihan kepercayaan di antara pemangku kepentingan.

## Mitigasi Penahanan Peristiwa dan Resolusi
- **Penahanan dan Mitigasi Insiden:** Tindakan segera untuk mengisolasi dan meminimalkan kerusakan, penyebaran, serta dampak.
- **Manajemen Kerentanan:** Memitigasi atau mendokumentasikan kerentanan baru mengurangi probabilitas keberhasilan serangan ransomware.

## Peningkatan Respon Melalui Pembelajaran
- **Menggabungkan dan Memperbarui Rencana serta Strategi Respons:** Menggabungkan pelajaran yang didapat meminimalkan keberhasilan serangan ransomware di masa depan dan memulihkan kepercayaan pemangku kepentingan.

## Eksekusi dan Pemeliharaan Proses Pemulihan
- **Eksekusi Rencana Pemulihan:** Inisiasi segera setelah identifikasi akar masalah dapat memotong kerugian.
- **Menggabungkan dan Memperbarui Strategi Pemulihan:** Menggabungkan pelajaran yang didapat meminimalkan serangan yang berhasil di masa depan dan mempertahankan efektivitas perencanaan kontingensi.

## Koordinasi Pemulihan dengan Pihak Internal dan Eksternal
- **Mengelola Hubungan Masyarakat dan Perbaikan Reputasi:** Meminimalkan dampak bisnis dan memulihkan kepercayaan di antara pemangku kepentingan.
- **Mengomunikasikan Kegiatan Pemulihan:** Komunikasi dengan pemangku kepentingan internal dan eksternal meminimalkan dampak bisnis dan memulihkan kepercayaan.
