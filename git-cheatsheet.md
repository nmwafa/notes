# Git Cheatsheet

## Daftar Isi

1. [Pengantar dan Konfigurasi](#1-pengantar-dan-konfigurasi)
2. [Perintah Dasar (Mulai Repositori & Snapshot)](#2-perintah-dasar-mulai-repositori--snapshot)
3. [Melihat Riwayat dan Perubahan](#3-melihat-riwayat-dan-perubahan)
4. [Membatalkan Perubahan & Memperbaiki Kesalahan](#4-membatalkan-perubahan--memperbaiki-kesalahan)
5. [Branching & Merging](#5-branching--merging)
6. [Bekerja dengan Remote (GitHub/GitLab)](#6-bekerja-dengan-remote-githubgitlab)
7. [Stashing & Pembersihan](#7-stashing--pembersihan)
8. [Rebasing Interaktif & Rewriting History](#8-rebasing-interaktif--rewriting-history)
9. [Git Worktree & Submodules](#9-git-worktree--submodules)
10. [Debugging (Bisect, Blame, Log filtering)](#10-debugging-bisect-blame-log-filtering)
11. [Git Hooks & Automatisasi](#11-git-hooks--automatisasi)
12. [Perintah Lanjutan Lainnya (Reflog, Archive, Patch)](#12-perintah-lanjutan-lainnya-reflog-archive-patch)

---

## 1. Pengantar dan Konfigurasi

```bash
# Menampilkan versi Git
git --version

# Mengatur nama dan email global (wajib untuk commit)
git config --global user.name "Nama Anda"
git config --global user.email "email@example.com"

# Mengatur editor default (contoh: vim, nano, code)
git config --global core.editor "code --wait"

# Melihat semua konfigurasi
git config --list

# Menyimpan kredential (username/password/token) untuk sementara
git config --global credential.helper cache

# Alias (shortcut) contoh: git st = git status
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.unstage 'reset HEAD --'

# Mengabaikan file/folder tertentu -> buat file .gitignore
# Contoh isi .gitignore:
# node_modules/
# *.log
# .env
```

## 2. Perintah Dasar (Mulai Repositori & Snapshot)

```bash
# Inisialisasi repositori baru di folder saat ini
git init

# Clone repositori dari remote (HTTPS atau SSH)
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git

# Melihat status (file yang berubah, staged, untracked)
git status
git status -s  # short status

# Menambahkan file ke staging area
git add file.txt          # satu file
git add .                 # semua perubahan (new, modified, deleted)
git add *.js              # semua file dengan ekstensi .js

# Menghapus file dari staging area (tidak jadi commit)
git reset HEAD file.txt

# Melakukan commit
git commit -m "Pesan commit singkat"
git commit -am "Langsung add + commit untuk file yang sudah tracked"

# Mengubah pesan commit terakhir
git commit --amend -m "Pesan baru"

# Melihat daftar commit (log)
git log
git log --oneline         # satu baris per commit
git log --graph --oneline --decorate  # visual branch
```

## 3. Melihat Riwayat dan Perubahan

```bash
# Perbedaan antara working directory dan staging area
git diff

# Perbedaan staging area vs commit terakhir
git diff --staged
git diff --cached

# Perbedaan antara dua commit (hash1 dan hash2)
git diff hash1 hash2

# Perbedaan antara branch A dan branch B
git diff branchA..branchB

# Menampilkan siapa yang mengubah baris tertentu di file
git blame file.txt

# Melihat riwayat perubahan suatu file (termasuk rename)
git log --follow -p file.txt

# Melihat commit dalam rentang waktu
git log --since="2 weeks ago"
git log --until="2024-01-01"
git log --grep="bug fix"   # cari pesan commit
git log -S"fungsi"          # cari teks dalam perubahan kode
```

## 4. Membatalkan Perubahan & Memperbaiki Kesalahan

```bash
# Membuang perubahan di working directory (belum di add)
git checkout -- file.txt
# Atau (Git 2.23+)
git restore file.txt

# Unstage file (kembali ke modified setelah add)
git reset HEAD file.txt
# Atau
git restore --staged file.txt

# Membatalkan commit TERAKHIR tapi tetap menyimpan perubahannya
git reset --soft HEAD~1   # perubahan di staging
git reset --mixed HEAD~1  # perubahan di working (default)
git reset --hard HEAD~1   # BUANG PERUBAHAN (hati-hati!)

# Membatalkan commit dan membuat commit baru untuk membalikkan (revert)
git revert HEAD           # buat commit kebalikan dari commit terakhir
git revert hashCommit

# Mengembalikan file ke versi commit tertentu
git restore --source=hashCommit file.txt
```

## 5. Branching & Merging

```bash
# Melihat semua branch lokal
git branch
git branch -v            # dengan commit terakhir

# Membuat branch baru
git branch nama-branch

# Beralih ke branch
git checkout nama-branch
git switch nama-branch   # alternatif modern

# Membuat dan langsung beralih
git checkout -b nama-branch
git switch -c nama-branch

# Menghapus branch (sudah di-merge)
git branch -d nama-branch

# Memaksa hapus branch (meski belum di-merge)
git branch -D nama-branch

# Merge branch lain ke branch saat ini
git merge branch-sumber

# Menyelesaikan konflik merge:
# 1. Edit file yang konflik
# 2. git add .
# 3. git commit

# Menampilkan branch yang sudah di-merge
git branch --merged
git branch --no-merged
```

## 6. Bekerja dengan Remote (GitHub/GitLab)

```bash
# Menambahkan remote origin
git remote add origin https://github.com/user/repo.git

# Melihat daftar remote
git remote -v

# Mengambil perubahan dari remote (tanpa merge)
git fetch origin
git fetch --all

# Mengambil dan langsung merge ke branch lokal (pull)
git pull origin main
git pull --rebase origin main   # pull dengan rebase

# Mengirim commit ke remote (push)
git push origin main
git push -u origin main         # set upstream, selanjutnya cukup git push

# Menghapus branch di remote
git push origin --delete nama-branch

# Melihat branch remote
git branch -r
git branch -a                   # semua termasuk lokal

# Clone repositori dengan semua remote branch
git clone --mirror url

# Mengubah URL remote
git remote set-url origin url-baru
```

## 7. Stashing & Pembersihan

```bash
# Menyimpan perubahan sementara (stash)
git stash
git stash push -m "pesan stash"

# Melihat daftar stash
git stash list

# Menerapkan stash terakhir dan menghapusnya dari daftar
git stash pop

# Menerapkan stash tanpa menghapus
git stash apply stash@{2}

# Menghapus stash tertentu
git stash drop stash@{0}

# Menghapus semua stash
git stash clear

# Membuat branch dari stash
git stash branch nama-branch

# Membersihkan file untracked
git clean -n            # preview apa yang akan dihapus
git clean -f            # hapus file untracked
git clean -fd           # hapus juga folder untracked
git clean -fx           # hapus termasuk yang di .gitignore
```

## 8. Rebasing Interaktif & Rewriting History

```bash
# Rebase branch saat ini ke branch lain (misal main)
git rebase main

# Rebase interaktif untuk 3 commit terakhir
git rebase -i HEAD~3

# Di editor rebase, perintah yang umum:
# pick   -> gunakan commit
# reword -> gunakan tapi ubah pesan
# edit   -> berhenti untuk mengubah
# squash -> gabung dengan commit sebelumnya
# drop   -> hapus commit

# Mengubah author commit terakhir
git commit --amend --author="Nama <email>"

# Mengubah author untuk beberapa commit (rebase -i, lalu di setiap commit pilih edit)
git rebase -i HEAD~5
# lalu untuk setiap commit: git commit --amend --author="..." --no-edit && git rebase --continue

# Filter branch secara masal (misal mengganti email)
git filter-branch --env-filter '
if [ "$GIT_AUTHOR_EMAIL" = "lama@example.com" ]; then
    GIT_AUTHOR_EMAIL="baru@example.com"
fi' -- --all

# Alternatif modern: git filter-repo (perlu install terpisah)
```

## 9. Git Worktree & Submodules

```bash
# WORKTREE: memungkinkan checkout multiple branch dalam folder berbeda
git worktree add ../folder-baru nama-branch
git worktree list
git worktree remove ../folder-baru

# SUBMODULE: repositori di dalam repositori
git submodule add https://github.com/user/lib.git libs/lib
git submodule update --init --recursive   # clone semua submodule
git submodule update --remote             # update submodule ke commit terbaru

# Menghapus submodule (manual multi-langkah)
git submodule deinit -f -- libs/lib
rm -rf .git/modules/libs/lib
git rm -f libs/lib
```

## 10. Debugging (Bisect, Blame, Log filtering)

```bash
# BISECT: mencari commit yang menyebabkan bug
git bisect start
git bisect bad HEAD          # commit saat ini bug
git bisect good hashCommit   # commit yang masih baik
# Git akan checkout commit tengah, test, lalu beri tahu:
git bisect good              # jika commit ini baik
git bisect bad               # jika commit ini buruk
# Ulangi sampai ketemu
git bisect reset             # selesai, kembali ke branch semula

# BLAME dengan rentang baris
git blame -L 10,20 file.txt

# Melihat log dengan filter path
git log -- file.txt
git log --stat                # statistik perubahan file

# Mencari commit berdasarkan konten patch (string)
git log -S"kata kunci"

# Mencari berdasarkan regex dalam perubahan
git log -G"regex"
```

## 11. Git Hooks & Automatisasi

Hooks terletak di `.git/hooks/`. Aktifkan dengan menghapus ekstensi `.sample`.

```bash
# Contoh pre-commit: cek kesalahan syntax
# Buat file .git/hooks/pre-commit
#!/bin/sh
# npm run lint
# jika gagal, exit 1

# commit-msg: memastikan format pesan commit
#!/bin/sh
# commit_regex='^(feat|fix|docs): '
# if ! grep -qE "$commit_regex" "$1"; then
#     echo "Format pesan salah"
#     exit 1
# fi

# post-commit: notifikasi atau trigger deploy
#!/bin/sh
# curl -X POST https://webhook.example.com/deploy

# Pastikan file hook executable
chmod +x .git/hooks/pre-commit
```

## 12. Perintah Lanjutan Lainnya (Reflog, Archive, Patch)

```bash
# REFLOG: catatan semua pergerakan HEAD (menyelamatkan commit yang hilang)
git reflog
git checkout HEAD@{2}        # kembali ke posisi HEAD 2 langkah lalu
git reflog expire --expire=now --all && git gc --prune=now  # bersihkan reflog

# ARCHIVE: membuat zip/tar dari commit tertentu tanpa riwayat .git
git archive --format=zip --output=proyek.zip HEAD
git archive --format=tar.gz HEAD > proyek.tar.gz

# PATCH: membuat file diff untuk dikirim via email
git format-patch -1 HEAD           # patch untuk commit terakhir
git format-patch main..feature     # semua commit di feature belum di main
git apply patch-file.patch         # menerapkan patch

# GIT GREP: mencari teks dalam file yang di-track (lebih cepat dari grep biasa)
git grep "function_name"
git grep -n "TODO"                  # dengan nomor baris
git grep -e "regex" -- "*.js"       # hanya file js

# GIT NOTES: menambahkan catatan ke commit tanpa mengubah hash
git notes add -m "Catatan tambahan" hashCommit
git notes show hashCommit
git notes push origin refs/notes/*   # sinkronisasi notes ke remote

# SPARSE CHECKOUT: clone hanya subfolder tertentu
git clone --no-checkout https://github.com/user/repo.git
cd repo
git sparse-checkout init --cone
git sparse-checkout set folder/subfolder
git checkout main
```

---

> **Tips Keamanan:**
> - Jangan pernah gunakan `git reset --hard` jika tidak yakin. Gunakan `git stash` atau `git branch backup` dulu.
> - Simpan token akses pribadi (PAT) dengan aman. Jangan commit credential.
> - Untuk proyek besar, gunakan `git gc` (garbage collection) untuk mengoptimalkan ukuran repositori.
