# TMUX Cheatsheet

>Hampir semua *shortcut* di dalam tmux membutuhkan **Prefix Key** terlebih dahulu.
>
>Secara *default*, prefix key untuk tmux adalah: `Ctrl + b` (tekan Ctrl dan b bersamaan, lalu lepas, baru tekan tombol *shortcut* berikutnya).

---

### Manajemen Sesi (Sessions)
Dijalankan di terminal **sebelum** masuk ke tmux (atau di luar tmux):

- `tmux` : Memulai sesi tmux baru.
- `tmux new -s [nama]` : Memulai sesi baru dengan nama tertentu.
- `tmux ls` : Melihat daftar sesi tmux yang sedang berjalan.
- `tmux attach` atau `tmux a` : Masuk kembali (*attach*) ke sesi terakhir.
- `tmux a -t [nama]` : Masuk kembali ke sesi dengan nama tertentu.
- `tmux kill-session -t [nama]` : Menghapus/mematikan sesi tertentu.

Shortcut di dalam sesi (Gunakan `Ctrl + b` lalu tekan):
- `d` : *Detach* (keluar dari sesi saat ini, tapi membiarkannya tetap berjalan di *background*).
- `s` : Menampilkan daftar sesi interaktif untuk berpindah.
- `$` : Mengganti nama sesi saat ini.

---

### Manajemen Jendela (Windows)
*Window* di tmux mirip seperti *tab* pada browser.

Shortcut (Gunakan `Ctrl + b` lalu tekan):
- `c` : Membuat window baru (*create*).
- `p` : Pindah ke window sebelumnya (*previous*).
- `n` : Pindah ke window selanjutnya (*next*).
- `0` - `9` : Pindah ke window berdasarkan nomor.
- `w` : Menampilkan daftar window interaktif.
- `,` : Mengganti nama window saat ini.
- `&` : Menutup window saat ini (akan ada konfirmasi `y/n`).

---

### Manajemen Panel (Panes)
*Pane* digunakan untuk membagi satu window menjadi beberapa bagian (split screen).

Shortcut (Gunakan `Ctrl + b` lalu tekan):
- `%` : Membagi layar secara vertikal (kiri dan kanan).
- `"` : Membagi layar secara horizontal (atas dan bawah).
- `Panah (← ↑ → ↓)` : Berpindah antar panel.
- `x` : Menutup panel saat ini (akan ada konfirmasi `y/n`).
- `z` : *Zoom-in* panel saat ini menjadi *fullscreen* (tekan `Ctrl + b` lalu `z` lagi untuk *zoom-out*).
- `q` : Menampilkan nomor panel (tekan nomor yang muncul untuk pindah ke panel tersebut dengan cepat).
- `{` atau `}` : Menukar posisi panel saat ini dengan panel di sebelahnya.
- `Tahan Ctrl + Panah` : Mengubah ukuran panel (*resize*). *(Catatan: tekan terus Ctrl, lalu tekan b, lalu tekan panah)*.

---

### Copy Mode & Scroll
Secara default tidak bisa menggunakan *scroll mouse* untuk melihat log ke atas di tmux. Harus masuk ke *Copy Mode*.

Shortcut (Gunakan `Ctrl + b` lalu tekan):
- `[` : Masuk ke *Copy Mode*.
  - Setelah masuk, gunakan tanda panah (`↑` / `↓`), `Page Up`, atau `Page Down` untuk *scroll* terminal.
  - Tekan **`q`** untuk keluar dari mode ini.
- `?` : Menampilkan daftar semua *shortcut* / *keybindings* tmux (tekan `q` untuk keluar).

---
**Tips Tambahan:** Jika merasa `Ctrl + b` kurang nyaman karena terlalu jauh, ubah *prefix* menjadi `Ctrl + a` (mirip GNU Screen) dengan mengedit file konfigurasi `~/.tmux.conf`.
