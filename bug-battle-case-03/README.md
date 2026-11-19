# Kasus 03 — Pencarian dan Filter Buku

**Level:** Menengah–Lanjut
**Fokus:** Request, query string, Eloquent, `where`, `orWhere`, `LIKE`, filter kategori, Blade.

Aplikasi Laravel yang menampilkan daftar buku dengan fitur pencarian berdasarkan judul atau penulis dan filter berdasarkan kategori.

## Fitur

- Pencarian parsial pada judul **atau** penulis (memakai `LIKE`)
- Filter berdasarkan kategori
- Pencarian dan filter dapat digabung (kondisi AND)
- Input pencarian dan pilihan kategori dipertahankan setelah submit
- Pesan "Buku tidak ditemukan." saat hasil kosong
- Tombol Reset untuk mengosongkan pencarian dan filter

## Struktur Data

Tabel `categories`:

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | bigint | Primary key |
| `name` | string | Nama kategori (unik) |
| `created_at`, `updated_at` | timestamp | Dibuat otomatis oleh Laravel |

Tabel `books`:

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | bigint | Primary key |
| `title` | string | Judul buku |
| `author` | string | Penulis |
| `category_id` | bigint (foreign key) | Merujuk ke `categories.id`, `cascadeOnDelete` |
| `created_at`, `updated_at` | timestamp | Dibuat otomatis oleh Laravel |

Data awal dari `DatabaseSeeder`: 3 kategori (Teknologi, Akuntansi, Manajemen) dan 6 buku.

| Judul | Penulis | Kategori |
|---|---|---|
| Laravel untuk Pemula | Andi | Teknologi |
| Belajar Python | Budi | Teknologi |
| Pemrograman Web Modern | Citra | Teknologi |
| Pengantar Akuntansi | Dewi | Akuntansi |
| Akuntansi Biaya | Eko | Akuntansi |
| Manajemen Keuangan | Fitri | Manajemen |

## Routing

| Method | URL | Controller@method | Fungsi |
|---|---|---|---|
| GET | `/` | — | Redirect ke `/books` |
| GET | `/books` | `BookController@index` | Daftar, pencarian, dan filter buku |

Parameter query string pada `/books`:

| Parameter | Fungsi |
|---|---|
| `keyword` | Kata kunci untuk judul atau penulis |
| `category` | ID kategori untuk filter |

Contoh: `/books?keyword=lara&category=1`

## Logika Pencarian

Pada `BookController@index`, kondisi diterapkan dengan `when()` sehingga hanya aktif jika parameternya diisi:

```php
$books = Book::with('category')
    ->when($keyword, function ($query) use ($keyword) {
        $query->where(function ($q) use ($keyword) {
            $q->where('title', 'like', "%{$keyword}%")
              ->orWhere('author', 'like', "%{$keyword}%");
        });
    })
    ->when($request->category, function ($query) use ($request) {
        $query->where('category_id', $request->category);
    })
    ->get();
```

Kondisi `where(...)->orWhere(...)` untuk judul dan penulis dibungkus dalam satu closure. Pembungkus ini penting agar `orWhere` tidak merusak filter kategori.

## Instalasi

1. Buat proyek Laravel baru:
   ```bash
   composer create-project laravel/laravel bug-battle-case-03
   ```
2. Salin folder `app`, `database`, `resources`, dan `routes` dari folder kasus ini ke proyek (timpa jika diminta).
3. Buat database `bug_battle_case_03` di MySQL.
4. Atur `.env`:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=bug_battle_case_03
   DB_USERNAME=root
   DB_PASSWORD=
   ```
5. Jalankan migrasi dan seeder lebih dulu, baru bersihkan cache (Laravel baru memakai cache `database`, sehingga tabel `cache` harus ada):
   ```bash
   php artisan migrate
   php artisan db:seed
   php artisan optimize:clear
   php artisan serve
   ```
6. Buka `http://127.0.0.1:8000/books` (atau `http://bug-battle-case-03.test/books` jika memakai Laragon).

> Pada kasus ini cukup `php artisan db:seed` (tanpa `--class`), karena `DatabaseSeeder` sudah berisi kategori dan buku.

## Gejala yang Diselidiki dan Hasil Pemeriksaan

| No | Gejala | Area yang diperiksa | Kondisi |
|---|---|---|---|
| 1 | Kata kunci dari form tidak terbaca oleh Controller | Nama input vs parameter di controller | Input bernama `keyword`; controller membaca `$request->keyword` |
| 2 | Pencarian hanya menemukan kecocokan persis | Operator query | Memakai `like` dengan `%{$keyword}%`, sehingga pencarian parsial |
| 3 | Filter kategori tidak bekerja | Nama select vs parameter, kondisi `where` | Select bernama `category`; controller memakai `when($request->category)` dengan `where('category_id', ...)` |
| 4 | Variabel hasil pencarian tidak tersedia di Blade | `compact()` vs variabel di view | `compact('books', 'categories')` sesuai dengan `$books` dan `$categories` di Blade |
| 5 | Input pencarian tidak dipertahankan setelah submit | Atribut `value` dan `selected` | `value="{{ request('keyword') }}"` dan `selected` pada kategori yang dipilih |

Kondisi tambahan: kondisi judul dan penulis digabung dengan benar dalam satu closure, dan pesan hasil kosong ditampilkan lewat `@forelse ... @empty`.

## Skenario Pengujian

| No | Skenario | Hasil yang diharapkan |
|---|---|---|
| 1 | Buka `/books` tanpa filter | Muncul 6 buku |
| 2 | Keyword `lara` | Hanya "Laravel untuk Pemula" (pencarian parsial) |
| 3 | Keyword `Citra` (nama penulis) | Muncul "Pemrograman Web Modern" |
| 4 | Kategori Akuntansi saja | Muncul 2 buku (Pengantar Akuntansi, Akuntansi Biaya) |
| 5 | Keyword `akuntansi` + kategori Teknologi | Hasil kosong dengan pesan "Buku tidak ditemukan." |
| 6 | Setelah submit, periksa form | Keyword dan kategori masih terisi; tombol Reset mengosongkannya |

## Catatan

- Hasil pemeriksaan di atas didasarkan pada pembacaan kode versi yang dikerjakan. Ubah kolom **Kondisi** dan lampirkan screenshot pengujian sesuai hasil yang kamu dapatkan saat menjalankan aplikasi.
- Jika ada bug yang ditemukan pada versi awal, tambahkan tabel berikut:

| No | File | Area | Gejala | Penyebab | Solusi |
|---|---|---|---|---|---|
| | | | | | |