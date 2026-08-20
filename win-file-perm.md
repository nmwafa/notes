---
title: "Windows File Permissions & Special Permissions"
layout: default
---

# Windows File Permissions & Special Permissions

---

## Daftar Isi
1. [Pengantar Sistem Perizinan Windows](#1-pengantar-sistem-perizinan-windows)
2. [NTFS Permissions vs Share Permissions](#2-ntfs-permissions-vs-share-permissions)
3. [Standar NTFS Permissions (Standard Permissions)](#3-standar-ntfs-permissions-standard-permissions)
4. [Special Permissions (Izin Khusus / Tingkat Lanjut)](#4-special-permissions-izin-khusus--tingkat-lanjut)
5. [Pewarisan Izin (Permission Inheritance & Propagation)](#5-pewarisan-izin-permission-inheritance--propagation)
6. [Aturan Evaluasi Izin (Effective Permissions & Rule of Precedence)](#6-aturan-evaluasi-izin-effective-permissions--rule-of-precedence)
7. [Manajemen Izin via CLI (ICACLS & PowerShell)](#7-manajemen-izin-via-cli-icacls--powershell)
8. [Praktik Terbaik (Best Practices)](#8-praktik-terbaik-best-practices)

---

## 1. Pengantar Sistem Perizinan Windows

Pada lingkungan Windows (khususnya Windows Server dan edisi Enterprise/Pro), keamanan data dikelola menggunakan **Access Control Model**. Setiap objek di sistem operasi (file, folder, registry key, printer) memiliki **Security Descriptor** yang berisi:
- **Owner (Pemilik Objek):** Pengguna yang memiliki hak inheren untuk mengubah izin.
- **DACL (Discretionary Access Control List):** Daftar entitas keamanan (*users/groups*) beserta hak akses yang diizinkan (*Allow*) atau ditolak (*Deny*). Setiap entri disebut **ACE (Access Control Entry)**.
- **SACL (System Access Control List):** Digunakan untuk keperluan audit dan logging akses objek ke Windows Event Log.

---

## 2. NTFS Permissions vs Share Permissions

Ketika file/folder dibagikan melalui jaringan (*network share*), terdapat dua lapisan izin yang berlaku:

| Kategori | Share Permissions | NTFS Permissions |
| :--- | :--- | :--- |
| **Cakupan Akses** | Hanya berlaku saat folder diakses melalui jaringan (`\\Server\Share`). | Berlaku baik saat diakses lokal maupun lewat jaringan. |
| **Sistem Berkas** | Berlaku pada format FAT32, exFAT, maupun NTFS. | Hanya berlaku pada partisi yang diformat dengan sistem NTFS atau ReFS. |
| **Granularitas** | Terbatas pada 3 tingkat (*Read, Change, Full Control*). | Sangat detail (*Standard* hingga 14 *Special Permissions*). |
| **Tingkat Objek** | Hanya berlaku pada tingkat folder yang di-*share*. | Berlaku hingga tingkat individual file dan subfolder. |

> **Prinsip Kombinasi (Most Restrictive):**  
> Jika sebuah resource diakses via jaringan, izin akhir yang berlaku (**Effective Permission**) adalah kombinasi **paling ketat / paling membatasi** (*most restrictive*) antara Share Permission dan NTFS Permission.

---

## 3. Standar NTFS Permissions (Standard Permissions)

Standar perizinan adalah kumpulan preset dari beberapa *Special Permissions* yang dirancang untuk mempermudah konfigurasi umum.

| Izin Standar (*Basic*) | Deskripsi & Kemampuan Akses |
| :--- | :--- |
| **Read (Membaca)** | Membaca isi file, melihat atribut file/folder, melihat perizinan keamanan. |
| **Write (Menulis)** | Menulis/menimpa data ke file, membuat file/subfolder baru dalam folder, mengubah atribut standar. |
| **Read & Execute** | Memiliki seluruh hak *Read* ditambah kemampuan untuk mengeksekusi file program/skrip (`.exe`, `.bat`, dll) dan menelusuri folder (*Traverse Folder*). |
| **List Folder Contents** | Menampilkan daftar file dan subfolder di dalam direktori (hanya berlaku untuk objek folder). |
| **Modify (Mengubah)** | Menggabungkan *Read & Execute* + *Write* + hak untuk **menghapus (*Delete*)** file atau folder. Tidak dapat mengubah kepemilikan atau hak izin. |
| **Full Control** | Hak akses tanpa batas. Termasuk semua hak di atas ditambah kemampuan untuk **mengubah izin (*Change Permissions*)** dan **mengambil alih kepemilikan (*Take Ownership*)**. |

---

## 4. Special Permissions (Izin Khusus / Tingkat Lanjut)

Windows membagi kontrol akses menjadi **14 izin khusus individual**. Izin inilah yang membentuk *Standard Permissions*.

Berikut rincian lengkap ke-14 *Special Permissions*:

### 1. Traverse Folder / Execute File
- **Traverse Folder (Folder):** Mengizinkan pengguna berpindah melewati folder tersebut untuk mengakses file/subfolder di dalamnya, meskipun pengguna tidak memiliki izin *List Folder* pada folder tersebut.
- **Execute File (File):** Mengizinkan pengguna menjalankan program/skrip biner.

### 2. List Folder / Read Data
- **List Folder (Folder):** Mengizinkan pengguna melihat daftar nama file dan subfolder di dalam folder.
- **Read Data (File):** Mengizinkan pengguna membuka dan melihat isi teks/data dari file.

### 3. Read Attributes
- Mengizinkan melihat atribut dasar berkas/folder yang ditentukan oleh sistem (seperti atribut `Read-Only`, `Hidden`, `Archive`, atau `System`).

### 4. Read Extended Attributes (Read EA)
- Mengizinkan pembacaan atribut lanjutan berkas (*Extended Attributes*), yang biasanya didefinisikan oleh aplikasi khusus atau metadata tambahan program.

### 5. Create Files / Write Data
- **Create Files (Folder):** Mengizinkan pembuatan file baru di dalam direktori.
- **Write Data (File):** Mengizinkan penulisan perubahan atau penimpaan (*overwrite*) isi file.

### 6. Create Folders / Append Data
- **Create Folders (Folder):** Mengizinkan pembuatan subdirektori baru di dalam folder.
- **Append Data (File):** Mengizinkan penambahan data baru di akhir file tanpa menimpa atau memodifikasi data yang sudah ada sebelumnya (berguna untuk file *log*).

### 7. Write Attributes
- Mengizinkan pengubahan atribut dasar file/folder (misalnya memberi atau menghapus centang `Read-Only` atau `Hidden`).

### 8. Write Extended Attributes (Write EA)
- Mengizinkan penulisan atau modifikasi metadata dan atribut lanjutan (*Extended Attributes*) berkas.

### 9. Delete Subfolders and Files
- Mengizinkan penghapusan seluruh subfolder dan file di dalam suatu folder, **meskipun** subfolder/file tersebut tidak memiliki izin *Delete* secara individual. Izin ini hanya ada di tingkat folder.

### 10. Delete
- Mengizinkan penghapusan langsung pada objek file atau folder yang bersangkutan.

### 11. Read Permissions
- Mengizinkan pengguna membaca daftar perizinan (DACL) pada file/folder tersebut untuk mengetahui siapa saja yang memiliki akses.

### 12. Change Permissions (Write DAC)
- Mengizinkan pengguna memodifikasi daftar izin keamanan (DACL). Pengguna dengan hak ini dapat memberikan atau mencabut akses pengguna lain.

### 13. Take Ownership (Write Owner)
- Mengizinkan pengguna mengambil alih kepemilikan (*Ownership*) atas objek tersebut. Pemilik baru secara otomatis memiliki hak untuk mengubah DACL.

### 14. Synchronize
- Mengizinkan *thread/process* menunggu status sinyal dari *file handle* (sinkronisasi I/O asinkron). Diaktifkan secara otomatis oleh Windows API.

---

### Pemetaan: Standar vs Special Permissions

| Special Permission | Read | Read & Exec | Write | Modify | Full Control |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Traverse Folder / Execute File | | ✓ | | ✓ | ✓ |
| List Folder / Read Data | ✓ | ✓ | | ✓ | ✓ |
| Read Attributes | ✓ | ✓ | | ✓ | ✓ |
| Read Extended Attributes | ✓ | ✓ | | ✓ | ✓ |
| Create Files / Write Data | | | ✓ | ✓ | ✓ |
| Create Folders / Append Data | | | ✓ | ✓ | ✓ |
| Write Attributes | | | ✓ | ✓ | ✓ |
| Write Extended Attributes | | | ✓ | ✓ | ✓ |
| Delete Subfolders and Files | | | | | ✓ |
| Delete | | | | ✓ | ✓ |
| Read Permissions | ✓ | ✓ | ✓ | ✓ | ✓ |
| Change Permissions | | | | | ✓ |
| Take Ownership | | | | | ✓ |
| Synchronize | ✓ | ✓ | ✓ | ✓ | ✓ |

---

## 5. Pewarisan Izin (Permission Inheritance & Propagation)

Secara default, subfolder dan file baru akan mewarisi (*inherit*) izin dari folder induknya (*Parent Folder*).

### Opsi Cakupan (Applies To / Propagation Scope):
1. **This folder only:** Izin hanya berlaku pada folder itu sendiri.
2. **This folder, subfolders and files:** Izin berlaku pada folder dan seluruh turunannya (default).
3. **This folder and subfolders:** Berlaku pada folder dan subdirektori, tidak pada file mandiri.
4. **This folder and files:** Berlaku pada folder dan file di dalamnya, tidak pada subdirektori turunan.
5. **Subfolders and files only:** Tidak berlaku pada folder target, melainkan hanya pada isi di dalamnya.

### Memutus Pewarisan (Disabling Inheritance):
Ketika mematikan inheritance melalui menu *Advanced Security Settings*, ada dua pilihan:
- **Convert inherited permissions into explicit permissions:** Menyalin semua izin induk menjadi izin langsung (*explicit*), sehingga dapat diedit secara independen.
- **Remove all inherited permissions:** Menghapus seluruh izin warisan, menyisakan objek tanpa izin kecuali ditambahkan manual.

---

## 6. Aturan Evaluasi Izin (Effective Permissions & Rule of Precedence)

Ketika seorang pengguna mengakses objek, Windows menghitung izin akhir berdasarkan urutan prioritas berikut:

```text
[Prioritas Tertinggi]
        │
        ▼
1. Explicit DENY          (Penolakan langsung pada objek)
        │
        ▼
2. Explicit ALLOW         (Pemberian izin langsung pada objek)
        │
        ▼
3. Inherited DENY         (Penolakan yang diwariskan dari folder induk terdekat)
        │
        ▼
4. Inherited ALLOW        (Pemberian izin yang diwariskan dari folder induk terdekat)
        │
        ▼
[Prioritas Terendah / Default: No Access]
```

### Prinsip Penting:
- **Akumulasi Grup (Group Accumulation):** Jika pengguna menjadi anggota Group A (*Read*) dan Group B (*Write*), maka izin NTFS yang diperoleh adalah **Read + Write**.
- **Deny Mengalahkan Allow:** Jika Group C memiliki hak *Explicit Deny*, maka pengguna ditolak meskipun memiliki *Explicit Allow* dari Group A.

---

## 7. Manajemen Izin via CLI (ICACLS & PowerShell)

### Menggunakan Command Prompt (`icacls`)

```cmd
:: Melihat izin pada file/folder
icacls "C:\Data\Finance"

:: Memberikan hak Modify kepada grup 'FinanceTeam'
icacls "C:\Data\Finance" /grant FinanceTeam:(OI)(CI)M

:: Memberikan hak Read-Only secara rekursif
icacls "C:\Data\Public" /grant Everyone:(OI)(CI)R /T

:: Menghapus izin spesifik untuk user 'johndoe'
icacls "C:\Data\Finance" /remove johndoe

:: Mematikan inheritance dan menyalin izin yang ada (Convert)
icacls "C:\Data\Secure" /inheritance:d

:: Mengambil alih kepemilikan folder
takeown /F "C:\Data\LockedFolder" /R /D Y
```

> **Catatan Flag Inheritance pada icacls:**
> - `(OI)` = *Object Inherit* (diwariskan ke file).
> - `(CI)` = *Container Inherit* (diwariskan ke subfolder).
> - `(IO)` = *Inherit Only* (tidak berlaku untuk folder itu sendiri).

---

### Menggunakan PowerShell (`Get-Acl` & `Set-Acl`)

```powershell
# 1. Menampilkan Access Control List (ACL)
Get-Acl -Path "C:\Data\Project" | Format-List

# 2. Memberikan hak Read & Execute kepada user tertentu
$path = "C:\Data\Project"
$acl = Get-Acl -Path $path
$permission = "CORP\JaneDoe", "ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission
$acl.SetAccessRule($accessRule)
Set-Acl -Path $path -AclObject $acl

# 3. Mematikan Inheritance (IsProtected = $true, PreserveInheritance = $true)
$acl = Get-Acl -Path "C:\Data\Confidential"
$acl.SetAccessRuleProtection($true, $true)
Set-Acl -Path "C:\Data\Confidential" -AclObject $acl
```

---

## 8. Praktik Terbaik (Best Practices)

1. **Gunakan Model AGDLP / AGUDLP:**  
   Masukkan **A**ccounts ke dalam **G**lobal Groups, masukkan Global Groups ke **D**omain **L**ocal Groups, lalu berikan **P**ermissions pada Domain Local Groups. Hindari memberikan izin langsung ke *individual user*.
2. **Terapkan Prinsip Least Privilege:**  
   Berikan hak minimal yang dibutuhkan pengguna untuk menyelesaikan tugasnya (misal: gunakan *Modify* daripada *Full Control*).
3. **Hindari Penggunaan 'Deny' Berlebihan:**  
   Gunakan manajemen grup (*Allow*) secara terstruktur daripada menggunakan aturan *Deny*, karena *Deny* dapat menimbulkan konflik yang sulit di-*troubleshoot*.
4. **Lindungi Izin Folder Root:**  
   Pertahankan pewarisan (*inheritance*) dari root share dan matikan pewarisan hanya pada folder yang benar-benar membutuhkan isolasi hak akses.
