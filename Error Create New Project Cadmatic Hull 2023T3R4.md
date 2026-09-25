# 🛠️ Laporan Troubleshooting — Cadmatic Hull 2023 T3R4

**Kategori:** Software Support / CAD-CAM (Cadmatic Hull)
**Status:** ✅ Resolved
**Tanggal Kejadian:** 24–25 September 2026

---

## 📋 Daftar Isi

- [Ringkasan](#-ringkasan)
- [Informasi Environment](#-informasi-environment)
- [Gejala Masalah](#-gejala-masalah)
- [Log Error](#-log-error)
- [Kronologi & Diskusi dengan IT Engineer](#-kronologi--diskusi-dengan-it-engineer)
- [Analisis Penyebab (Root Cause)](#-analisis-penyebab-root-cause)
- [Solusi](#-solusi)
- [Verifikasi](#-verifikasi)
- [Pelajaran & Rekomendasi](#-pelajaran--rekomendasi)

---

## 📝 Ringkasan

Saat membuat/membuka project **T233401.pms** pada **Cadmatic Hull 2023 T3R4**, muncul serangkaian error yang menyebabkan project gagal dibuat dan hull lines gagal di-*import*. Setelah ditelusuri, akar masalah berasal dari **path project yang tidak konsisten** antar file konfigurasi (ada yang memakai drive letter `D:`, ada yang memakai UNC path `\\nupcadout2\cmprojects`, dan ada yang memakai mapped drive `U:`). Ketidakkonsistenan ini menyebabkan Cadmatic Hull kehilangan hak akses (*access rights*) saat mencoba membuat file di lokasi tersebut.

**Solusi:** menyamakan (menyeragamkan) path di **3 file XML konfigurasi** agar semuanya mengarah ke mapped drive yang sama (`U:`).

---

## 💻 Informasi Environment

| Item | Detail |
|---|---|
| Aplikasi | Cadmatic Hull 2023 T3R4 (Eagle Editor) |
| Nama Project | `T233401.pms` |
| Server | `nupcadout2` |
| Mapped Drive | `U:` (`\\nupcadout2\cmprojects`) |
| Norms | `ncgn161` |
| Deskripsi Project | *Testing Cadmatic 23T3R4* |

---

## ⚠️ Gejala Masalah

Error muncul berurutan saat proses pembuatan/pembukaan project berlangsung:

### 1. Gagal membuat file update norms
> **Eagle** — *Error, unable to set new update.*
> An error occurred while creating the file
> `\\nupcadout2\cmprojects\Run\T233401.pms\Hull\norms\ncgn161\norms_info.cmd`.
> This could be caused by insufficient access rights, please contact your system administrator.

![Error 1](.Foto/WhatsApp%20Image%202026-09-25%20at%2009.11.30.jpeg)

### 2. Gagal copy model directory default
> **Error** — Error, unable to copy default modeldirectory (`%hullcentre%\mod2d.ncg`) to current modeldirectory (`..\norms\ncgn161\mod2d`).

Log terkait:
```
Warning: Unable to find norms 'ncgn161', using 'ncgn161' instead
Converting bevel and weld settings from norms 'ncgn161' into project settings
Error: Error: -NCGNORMS/settings.cmd not found
```

![Error 2](.Foto/WhatsApp%20Image%202026-09-25%20at%2009.11.30%20(1).jpeg)

### 3. Import hull lines gagal
> **Error** — Importing hull lines failed.

Log terkait:
```
Error: Failed to read project administration cache: failed reading cache file:
Cannot start reading from file C:\Users\...\AppData\...
Error:
Error: Update Hull database failed.
Error:
```

![Error 3](.Foto/WhatsApp%20Image%202026-09-25%20at%2009.11.31.jpeg)

### 4. Project gagal dibuat (final error)
> **Error** — There were errors creating this project.

Log terkait:
```
Invalid input: 'fr60.5'
unit_conv: Error converting 'fr60.5'
Error: Invalid input: 60.5
No database present yet in the ncgdb directory
```

![Error 4](./Foto/WhatsApp%20Image%202026-09-25%20at%2009.11.31%20(1).jpeg)

---

## 🧾 Log Error

Ringkasan seluruh baris error (digabungkan berurutan sesuai kemunculan):

```text
Error, unable to set new update.
An error occurred while creating the file "\\nupcadout2\cmprojects\Run\T233401.pms\Hull\norms\ncgn161\norms_info.cmd".
This could be caused by insufficient access rights, please contact your system administrator.

Warning: Unable to find norms 'ncgn161', using 'ncgn161' instead
Converting bevel and weld settings from norms 'ncgn161' into project settings
Error: Error: -NCGNORMS/settings.cmd not found

Error: Failed to read project administration cache: failed reading cache file: Cannot start reading from file C:\Users\...\AppData\...
Error: Update Hull database failed.

Invalid input: 'fr60.5'
unit_conv: Error converting 'fr60.5'
Error: Invalid input: 60.5
No database present yet in the ncgdb directory
There were errors creating this project.
```

---

## 💬 Kronologi & Diskusi dengan IT Engineer

Berikut rangkuman percakapan troubleshooting dengan IT Engineer:

1. Ditanyakan letak masalah → dikonfirmasi terjadi pada **drive / mapped location**.
2. Sebelumnya, path project **tidak konsisten**: sebagian menggunakan **nama server** (UNC path), sebagian menggunakan **drive `D:`**.
3. Ketidakkonsistenan inilah yang memicu masalah **access rights**.
4. IT Engineer melakukan perubahan agar **semua path diarahkan ke drive `U:`**.
5. Lokasi yang perlu diubah dikonfirmasi ada di **3 file XML**.
6. IT Engineer menekankan pentingnya **membuat backup file sebelum diubah**, karena file yang corrupt berpotensi tidak bisa diperbaiki.
7. IT Engineer juga membuatkan **local project** sebagai sarana uji coba/backup jika project lain tidak bisa diakses.
8. Dikonfirmasi ulang bahwa masalah **hanya ada pada 3 file XML** tersebut — dan tidak ada perubahan lain yang dilakukan oleh user sebelumnya.
9. Disebutkan juga kemungkinan lain: penggunaan **norms baru** turut berkontribusi pada masalah, namun penyebab utama tetap pada path di file XML.

---

## 🔍 Analisis Penyebab (Root Cause)

Root cause diverifikasi langsung dari isi file konfigurasi:

**`hcaprojects.xml`**
(`C:\ProgramData\Cadmatic\HCA\HCA_PAL_DES_Root\hcaprojects.xml`)

```xml
<HcaProjects>
  <Project>
    <ProjectPath>D:\CMProjects\Run\T233401.pms\Hull</ProjectPath>
    <CosHost>nupcadout2</CosHost>
    <CosPort>5052</CosPort>
  </Project>
  <Project>
    <ProjectPath>\\nupcadout2\cmprojects\Run\T233401_N.pms\Hull</ProjectPath>
    <CosHost>nupcadout2</CosHost>
    <CosPort>5052</CosPort>
  </Project>
</HcaProjects>
```

Terlihat jelas **dua format path berbeda** terdaftar untuk project sejenis:
- `D:\CMProjects\Run\...` → drive letter lokal/lama
- `\\nupcadout2\cmprojects\Run\...` → UNC path

Sementara itu, `projectlist.xml` (`U:\Hullcentre\hullcos\projectlist.xml`) dan `projectsiteinfo.xml` (`U:\Run\T233401.pms\Hull\administration\projectsiteinfo.xml`) justru menggunakan referensi **mapped drive `U:`**:

```xml
<PathInHull>U:\Run\T233401.pms</PathInHull>
```

```xml
<hullcentre>U:\hullcentre</hullcentre>
```

📌 **Kesimpulan Root Cause:**
Referensi path project **tidak seragam** di antara 3 file konfigurasi (`hcaprojects.xml`, `projectlist.xml`, `projectsiteinfo.xml`) — sebagian memakai drive letter (`D:`), sebagian memakai UNC path (`\\nupcadout2\...`), dan sebagian memakai mapped drive (`U:`). Karena hak akses (*access rights*) di server hanya diberikan secara konsisten untuk satu jenis path, Cadmatic Hull gagal membuat/menulis file (norms, database, cache) ketika path yang dipakai tidak sesuai dengan hak akses yang tersedia — sehingga muncul error berantai mulai dari update norms, import hull lines, hingga pembuatan project gagal total.

---

## ✅ Solusi

1. **Backup terlebih dahulu** ketiga file XML sebelum diedit, untuk menghindari file corrupt yang sulit dipulihkan.
2. Buka dan periksa file berikut, lalu **samakan seluruh path agar konsisten menggunakan mapped drive `U:`**:

   | No | File | Lokasi |
   |---|---|---|
   | 1 | `hcaprojects.xml` | `C:\ProgramData\Cadmatic\HCA\HCA_PAL_DES_Root\hcaprojects.xml` |
   | 2 | `projectlist.xml` | `U:\Hullcentre\hullcos\projectlist.xml` |
   | 3 | `projectsiteinfo.xml` | `U:\Run\<Project>.pms\Hull\administration\projectsiteinfo.xml` |

3. Pada `hcaprojects.xml`, ubah setiap `<ProjectPath>` yang masih menggunakan `D:\CMProjects\...` atau `\\nupcadout2\cmprojects\...` menjadi format yang konsisten, contoh:
   ```xml
   <ProjectPath>U:\Run\T233401.pms\Hull</ProjectPath>
   ```
4. Pastikan `<PathInHull>` pada `projectlist.xml` dan `<hullcentre>` pada `projectsiteinfo.xml` juga tetap merujuk ke drive `U:` (tidak dicampur dengan `D:` atau UNC path).
5. Simpan perubahan pada ketiga file.
6. Tutup dan buka ulang **Cadmatic Hull / Eagle Editor**.
7. Coba ulang proses pembukaan/pembuatan project.

---

## 🔎 Verifikasi

- ✅ Project dapat dibuat/dibuka tanpa error *"unable to set new update"*.
- ✅ Proses copy model directory (`mod2d.ncg`) berjalan normal, norms `ncgn161` terbaca.
- ✅ *Importing hull lines* berhasil tanpa error database.
- ✅ Tidak ada lagi pesan *"There were errors creating this project."*

---

## 💡 Pelajaran & Rekomendasi

- Selalu gunakan **satu format path yang konsisten** (disarankan mapped drive, misal `U:`) di seluruh file konfigurasi Cadmatic Hull, jangan mencampur drive letter lokal (`D:`) dengan UNC path server.
- Saat menambahkan project baru atau norms baru, verifikasi kembali ketiga file XML (`hcaprojects.xml`, `projectlist.xml`, `projectsiteinfo.xml`) agar tetap konsisten.
- **Selalu backup file konfigurasi** sebelum melakukan perubahan manual — file yang corrupt berpotensi tidak dapat diperbaiki.
- Jika ada perubahan norms/versi Cadmatic, catat dan cek kompatibilitasnya, karena hal ini juga sempat dicurigai sebagai kemungkinan penyebab tambahan.

---

*Dokumentasi ini dibuat berdasarkan hasil troubleshooting internal untuk referensi tim jika menemukan kasus serupa di kemudian hari.*
