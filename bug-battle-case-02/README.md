# Kasus 02 — Relasi Buku dan Kategori

**Level:** Menengah
**Fokus:** Migration, foreign key, Eloquent relationship (`belongsTo`, `hasMany`), eager loading.

Aplikasi Laravel yang menerapkan relasi one-to-many: satu kategori memiliki banyak buku, dan setiap buku dimiliki satu kategori.

## Fitur

- Menampilkan daftar buku beserta nama kategorinya
- Menambah buku baru dengan memilih kategori dari dropdown
- Validasi input, termasuk memastikan kategori yang dipilih memang ada
- Data awal kategori melalui seeder

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

Migrasi `categories` berjalan lebih dulu daripada `books` (urutan nama file), sehingga foreign key dapat dibuat.

## Relasi Model

| Model | Relasi | Keterangan |
|---|---|---|
| `Book` | `category()` → `belongsTo(Category::class)` | Setiap buku dimiliki satu kategori |
| `Category` | `books()` → `hasMany(Book::class)` | Satu kategori memiliki banyak buku |

## Routing

| Method | URL | Controller@method | Nama route | Fungsi |
|---|---|---|---|---|
| GET | `/` | — | — | Redirect ke `/books` |
| GET | `/books` | `BookController@index` | `books.index` | Daftar buku |
| GET | `/books/create` | `BookController@create` | `books.create` | Form tambah buku |
| POST | `/books` | `BookController@store` | `books.store` | Simpan buku baru |

## Validasi

Aturan pada `BookController@store`:

| Field | Aturan |
|---|---|
| `title` | wajib, teks, maksimal 255 karakter |
| `author` | wajib, teks, maksimal 255 karakter |
| `category_id` | wajib, harus ada di tabel `categories` (`exists:categories,id`) |

## Instalasi

1. Buat proyek Laravel baru:
   ```bash
   composer create-project laravel/laravel bug-battle-case-02
   ```
2. Salin folder `app`, `database`, `resources`, dan `routes` dari folder kasus ini ke proyek (timpa jika diminta).
3. Buat database `bug_battle_case_02` di MySQL.
4. Atur `.env`:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=bug_battle_case_02
   DB_USERNAME=root
   DB_PASSWORD=
   ```
5. Jalankan migrasi dan seeder lebih dulu, baru bersihkan cache (Laravel baru memakai cache `database`, sehingga tabel `cache` harus ada):
   ```bash
   php artisan migrate
   php artisan db:seed --class=CategorySeeder
   php artisan optimize:clear
   php artisan serve
   ```
6. Buka `http://127.0.0.1:8000/books` (atau `http://bug-battle-case-02.test/books` jika memakai Laragon).

> Seeder kategori wajib dijalankan. Jika tabel `categories` kosong, dropdown pada form kosong dan penyimpanan selalu gagal validasi.

## Gejala yang Diselidiki dan Hasil Pemeriksaan

| No | Gejala | Area yang diperiksa | Kondisi |
|---|---|---|---|
| 1 | Migrasi tabel buku gagal atau relasi tidak terjamin | Urutan migrasi, definisi `category_id` | Migrasi `categories` berjalan sebelum `books`; `foreignId('category_id')->constrained()->cascadeOnDelete()` menunjuk ke `categories.id` |
| 2 | Daftar buku gagal mengambil kategori | Eager loading di controller, nama method relasi di model | `Book::with('category')` sesuai dengan method `category()` di model |
| 3 | Nama kategori tidak tampil dengan benar | Pemanggilan relasi di Blade | Blade memakai `$book->category->name`, sesuai relasi `belongsTo` |
| 4 | Form tidak menampilkan pilihan kategori | Variabel dari `create()` vs Blade | `create()` mengirim `$categories`; Blade melakukan `@foreach ($categories ...)` dengan variabel yang sama |

## Skenario Pengujian

| No | Skenario | Hasil yang diharapkan |
|---|---|---|
| 1 | Buka `/books` saat data kosong | Halaman terbuka, tampil pesan bahwa belum ada buku |
| 2 | Buka form tambah buku | Dropdown berisi Akuntansi, Manajemen, Teknologi (urut abjad) |
| 3 | Tambah 3 buku: dua berkategori Teknologi, satu Akuntansi | Redirect ke `/books` dengan pesan sukses |
| 4 | Periksa kolom Kategori di `/books` | Ketiga buku menampilkan kategori yang benar |
| 5 | Simpan form dalam keadaan kosong | Muncul pesan validasi, data tidak tersimpan |
| 6 | Uji relasi lewat Tinker | Lihat perintah di bawah |

Uji relasi `hasMany` dan `belongsTo` lewat Tinker:

```bash
php artisan tinker
```

```php
App\Models\Category::where('name', 'Teknologi')->first()->books;
App\Models\Book::first()->category;
```

Kategori Teknologi harus mengembalikan 2 buku, dan buku pertama harus mengembalikan kategorinya.

## Catatan

- Hasil pemeriksaan di atas didasarkan pada pembacaan kode versi yang dikerjakan. Ubah kolom **Kondisi** dan lampirkan screenshot pengujian sesuai hasil yang kamu dapatkan saat menjalankan aplikasi.
- Jika ada bug yang ditemukan pada versi awal, tambahkan tabel berikut:

| No | File | Area | Gejala | Penyebab | Solusi |
|---|---|---|---|---|---|
| | | | | | |