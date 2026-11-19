# Kasus 01 — CRUD Manajemen Buku

**Level:** Mudah
**Fokus:** Route, Controller, Model, Migration, Blade, validasi dasar.

Aplikasi Laravel sederhana untuk mengelola data buku: melihat daftar, menambah, dan menghapus buku.

## Fitur

- Menampilkan daftar buku (terbaru di atas)
- Menambah buku baru dengan validasi
- Menghapus buku
- Pesan sukses setelah tambah/hapus

## Struktur Data

Tabel `books`:

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | bigint | Primary key |
| `title` | string | Judul buku |
| `author` | string | Penulis |
| `published_year` | unsigned small integer | Tahun terbit |
| `created_at`, `updated_at` | timestamp | Dibuat otomatis oleh Laravel |

## Routing

| Method | URL | Controller@method | Nama route | Fungsi |
|---|---|---|---|---|
| GET | `/` | — | — | Redirect ke `/books` |
| GET | `/books` | `BookController@index` | `books.index` | Daftar buku |
| GET | `/books/create` | `BookController@create` | `books.create` | Form tambah buku |
| POST | `/books` | `BookController@store` | `books.store` | Simpan buku baru |
| DELETE | `/books/{id}` | `BookController@destroy` | `books.destroy` | Hapus buku |

## Validasi

Aturan pada `BookController@store`:

| Field | Aturan |
|---|---|
| `title` | wajib, teks, maksimal 255 karakter |
| `author` | wajib, teks, maksimal 255 karakter |
| `published_year` | wajib, angka bulat, minimal 1000, maksimal tahun berjalan |

## Instalasi

1. Buat proyek Laravel baru:
   ```bash
   composer create-project laravel/laravel bug-battle-case-01
   ```
2. Salin folder `app`, `database`, `resources`, dan `routes` dari folder kasus ini ke proyek (timpa jika diminta).
3. Buat database `bug_battle_case_01` di MySQL.
4. Atur `.env`:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=bug_battle_case_01
   DB_USERNAME=root
   DB_PASSWORD=
   ```
5. Jalankan migrasi lebih dulu, baru bersihkan cache (Laravel baru memakai cache `database`, sehingga tabel `cache` harus ada):
   ```bash
   php artisan migrate
   php artisan optimize:clear
   php artisan serve
   ```
6. Buka `http://127.0.0.1:8000/books` (atau `http://bug-battle-case-01.test/books` jika memakai Laragon).

## Gejala yang Diselidiki dan Hasil Pemeriksaan

| No | Gejala | Area yang diperiksa | Kondisi |
|---|---|---|---|
| 1 | Halaman daftar buku tidak dapat dibuka | `routes/web.php`, method `index` di controller, view `books.index` | Route `/books` terhubung ke `BookController@index`; method dan view tersedia |
| 2 | Form tambah buku berperilaku tidak sesuai | Route `create`/`store`, `@csrf`, nama input vs aturan validasi | Route benar; form memakai `@csrf`; nama input (`title`, `author`, `published_year`) sama dengan aturan validasi |
| 3 | Data tidak tampil setelah disimpan | `$fillable` di model, variabel controller vs Blade | `$fillable` lengkap; `compact('books')` sesuai dengan `$books` di Blade |
| 4 | Tombol hapus tidak bekerja | Route `DELETE`, `@method('DELETE')`, method `destroy` | Route `DELETE /books/{id}`, `@method('DELETE')`, dan `destroy($id)` saling cocok |

## Skenario Pengujian

| No | Skenario | Hasil yang diharapkan |
|---|---|---|
| 1 | Buka `/books` saat data kosong | Halaman terbuka, tampil pesan bahwa belum ada buku |
| 2 | Tambah buku dengan data valid | Redirect ke `/books`, pesan "Buku berhasil ditambahkan.", buku muncul di daftar |
| 3 | Simpan form dalam keadaan kosong | Muncul pesan validasi, data tidak tersimpan |
| 4 | Isi tahun terbit di luar rentang (mis. `999` atau tahun depan) | Muncul pesan validasi pada `published_year` |
| 5 | Hapus salah satu buku | Redirect ke `/books`, pesan "Buku berhasil dihapus.", buku hilang dari daftar |

## Catatan

- Hasil pemeriksaan di atas didasarkan pada pembacaan kode versi yang dikerjakan. Isi kolom **Kondisi** dan lampirkan screenshot pengujian sesuai hasil yang kamu dapatkan saat menjalankan aplikasi.
- Jika ada bug yang ditemukan pada versi awal, tambahkan tabel berikut:

| No | File | Area | Gejala | Penyebab | Solusi |
|---|---|---|---|---|---|
| | | | | | |