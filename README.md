# E-Kos

**Cari Kos, Pesan Mudah, Tinggal Nyaman.**

Proyek mata kuliah **Pemrograman Web Dinamis 2026** oleh kelompok **Ngawiters**.

## 1. Identitas Kelompok

| Peran | Nama | NIM | Posisi |
| --- | --- | --- | --- |
| Ketua | Muchsin Hidayat Julianto | 2400018234 | Backend Developer |
| Anggota 1 | Dimas Pratama Anugraha | 2400018215 | Backend Developer |
| Anggota 2 | Nurrahmat Eko Nugroho | 2400018200 | Frontend Developer |
| Anggota 3 | Rafi Satya Prayoga | 2400018208 | Frontend Developer |

---

# Milestone 1

## 2. Rancangan Aplikasi Web

**Judul Aplikasi:** E-Kos

**Tagline:** Cari Kos, Pesan Mudah, Tinggal Nyaman.

**Deskripsi Singkat:** Kami membangun E-Kos sebagai aplikasi web untuk pengelolaan satu lokasi kos milik satu pemilik. Arsitekturnya kami pisahkan menjadi *backend* (REST API) dan *frontend* (SPA). Calon penyewa dapat melihat informasi kamar, mendaftar, mengunggah dokumen identitas, lalu menyewa kamar yang berstatus Tersedia secara langsung. Admin memverifikasi identitas dan pembayaran, mengelola kamar, serta memantau operasional kos. Pembayaran pada tahap ini berupa **simulasi** yang diverifikasi admin, sehingga tidak ada uang sungguhan yang berpindah.

**Fokus Milestone 1:** kami memfokuskan pekerjaan pada fondasi sistem, yaitu autentikasi dan hak akses (Admin, Calon Penyewa, Penyewa), pengelolaan data master kamar (tipe, kamar, harga, peraturan), katalog kamar publik, serta alur pengajuan dan verifikasi identitas.

**Stack Teknologi:**

- **Frontend:** React (JavaScript), Vite, Tailwind CSS.
- **Backend:** Node.js, Express.js (REST API), autentikasi JWT, hashing password dengan bcrypt.
- **Basis Data:** PostgreSQL dengan Prisma ORM.

---

## 3. Hak Akses Pengguna

Sistem kami memiliki dua peran akun, yaitu Admin dan Penyewa. Calon penyewa adalah akun terdaftar yang belum memiliki peran Penyewa, sehingga pada basis data kami bedakan menjadi tiga nilai `role`.

| Fitur (Milestone 1) | Pengunjung | Calon Penyewa | Penyewa | Admin |
| --- | --- | --- | --- | --- |
| Melihat katalog kamar, harga, dan peraturan | Ya | Ya | Ya | Ya |
| Registrasi dan login | Ya | Ya | Ya | Ya |
| Mengisi data identitas dan unggah dokumen | Tidak | Ya | Hanya jika diminta admin | Memeriksa |
| Verifikasi identitas | Tidak | Tidak | Tidak | Ya |
| CRUD tipe kamar, kamar, harga, dan peraturan | Tidak | Tidak | Tidak | Ya |
| Mengelola pengguna | Tidak | Tidak | Tidak | Ya |
| Melihat riwayat aktivitas (audit log) | Tidak | Tidak | Tidak | Ya |

Seluruh pemeriksaan hak akses kami lakukan di sisi server (middleware Express), bukan hanya di tampilan.

---

## 4. Desain Basis Data (ERD)

Kami memakai basis data relasional PostgreSQL yang dikelola dengan Prisma. Seluruh tabel di bawah ini adalah rancangan basis data untuk Milestone 1.

**Skema Relasi Dasar:**

- **users** (1) ---- (1) **identity_documents** *(satu akun memiliki satu data identitas yang melekat pada akun)*
- **identity_documents** (1) ---- (M) **identity_uploads** *(riwayat unggah dan unggah ulang dokumen)*
- **room_types** (1) ---- (M) **rooms** *(satu tipe memiliki banyak kamar)*
- **room_types** (1) ---- (M) **rental_prices** *(harga per tipe dan periode sewa)*
- **users** (1) ---- (M) **audit_logs** *(satu pengguna memiliki banyak riwayat tindakan)*
- **house_rules**: menyimpan versi peraturan kos

### 4.1 Struktur Tabel Milestone 1

**1. Tabel `users`** (Modul Autentikasi)

- `id` (UUID, Primary Key)
- `nama` (VARCHAR)
- `email` (VARCHAR, Unique)
- `password` (VARCHAR, hash bcrypt)
- `no_telepon` (VARCHAR)
- `role` (ENUM: `admin`, `calon_penyewa`, `penyewa`; default `calon_penyewa`)
- `status` (BOOLEAN, default true)
- `created_at`, `updated_at` (TIMESTAMP)

**2. Tabel `identity_documents`** (Modul Verifikasi Identitas)

- `id` (UUID, Primary Key)
- `user_id` (UUID, Foreign Key ke `users.id`, Unique)
- `nomor_identitas` (VARCHAR)
- `jenis_dokumen` (VARCHAR, contoh: KTP)
- `status` (ENUM: `belum_diunggah`, `menunggu_verifikasi`, `perlu_perbaikan`, `disetujui`, `ditolak_final`)
- `alasan` (TEXT, wajib saat perlu perbaikan atau ditolak final)
- `diverifikasi_oleh` (UUID, Foreign Key ke `users.id`, nullable)
- `diverifikasi_pada` (TIMESTAMP, nullable)
- `created_at`, `updated_at` (TIMESTAMP)

**3. Tabel `identity_uploads`** (Riwayat unggahan dokumen)

- `id` (UUID, Primary Key)
- `identity_document_id` (UUID, Foreign Key)
- `file_path` (VARCHAR, disimpan di folder privat, bukan folder publik)
- `created_at` (TIMESTAMP)

**4. Tabel `room_types`**

- `id` (UUID, Primary Key)
- `nama` (VARCHAR, contoh: Standar, AC)
- `deskripsi` (TEXT)
- `fasilitas` (TEXT/JSON)
- `created_at`, `updated_at` (TIMESTAMP)

**5. Tabel `rooms`**

- `id` (UUID, Primary Key)
- `room_type_id` (UUID, Foreign Key ke `room_types.id`)
- `nomor_kamar` (VARCHAR, Unique)
- `foto` (VARCHAR)
- `status` (ENUM: `tersedia`, `terkunci`, `terisi`, `dalam_perawatan`; default `tersedia`)
- `created_at`, `updated_at` (TIMESTAMP)

**6. Tabel `rental_prices`**

- `id` (UUID, Primary Key)
- `room_type_id` (UUID, Foreign Key)
- `periode_bulan` (INT: 1, 6, 12)
- `harga` (INT, Rupiah)
- `diskon` (INT, default 0)
- Unique gabungan: (`room_type_id`, `periode_bulan`)

**7. Tabel `house_rules`**

- `id` (UUID, Primary Key)
- `versi` (INT)
- `isi` (TEXT)
- `aktif` (BOOLEAN)
- `created_at` (TIMESTAMP)

**8. Tabel `audit_logs`** (Modul Audit)

- `id` (BIGINT, Primary Key)
- `user_id` (UUID, Foreign Key ke `users.id`)
- `action` (VARCHAR, contoh: `login`, `verify_identity`, `update_room_status`)
- `description` (TEXT)
- `ip_address` (VARCHAR)
- `created_at` (TIMESTAMP)

### 4.2 Aturan Integritas yang Sudah Dirancang Sejak Awal

1. Kamar memiliki status `terkunci` yang hanya boleh diubah oleh sistem, bukan admin secara manual.
2. Satu pengguna hanya memiliki satu data identitas; riwayat unggahan tersimpan terpisah.
3. Perubahan penting (verifikasi identitas, harga, status kamar) wajib tercatat di `audit_logs`.
4. Dokumen identitas disimpan di lokasi yang tidak dapat diakses publik dan hanya dilayani lewat endpoint yang terotorisasi.

---

## 5. Daftar Endpoint API (Rencana Milestone 1)

| Modul | Method dan Path | Akses |
| --- | --- | --- |
| Auth | `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me` | Publik / login |
| Katalog | `GET /api/rooms`, `GET /api/rooms/:id`, `GET /api/house-rules/active` | Publik |
| Tipe kamar | `GET/POST/PUT/DELETE /api/room-types` | Admin |
| Kamar | `POST/PUT/DELETE /api/rooms` | Admin |
| Harga | `GET/POST/PUT/DELETE /api/rental-prices` | Admin |
| Peraturan | `GET/POST/PUT /api/house-rules` | Admin |
| Identitas (pengguna) | `GET/POST /api/identity`, `POST /api/identity/reupload` | Calon penyewa |
| Identitas (admin) | `GET /api/admin/identities`, `PATCH /api/admin/identities/:id/review` | Admin |
| Pengguna | `GET/PUT/DELETE /api/users` | Admin |
| Audit | `GET /api/audit-logs` | Admin |

---

## 6. Rencana Pengerjaan dan Pembagian Tugas

| Anggota | Tanggung Jawab Milestone 1 |
| --- | --- |
| Muchsin (BE, Ketua) | Setup monorepo, autentikasi JWT dan middleware role, modul verifikasi identitas, audit log, koordinasi repositori |
| Dimas (BE) | Skema Prisma dan migrasi, seeder, CRUD tipe kamar, kamar, harga, dan peraturan, upload file |
| Nurrahmat (FE) | Setup Vite dan Tailwind, layout dan routing, halaman login dan registrasi, dashboard admin, halaman kelola kamar |
| Rafi (FE) | Katalog kamar publik dan detail kamar, formulir identitas dan unggah dokumen, halaman status identitas, kelola pengguna |

---

## 7. Repositori & Inisialisasi Proyek

**Tautan Repositori GitHub:** https://github.com/nurrahmat-eko/E-Kos

Kami mengundang dosen pengampu untuk akses repositori.

**Status Setup Lingkungan:**

- Repositori menggunakan format *Monorepo* (folder `backend` dan `frontend`).
- *Backend* menggunakan Express.js dan terhubung ke PostgreSQL melalui Prisma (`DATABASE_URL` disimpan di `.env`, tidak di-commit).
- *Frontend* diinisialisasi dengan Vite (React JavaScript) dan Tailwind CSS.

**Struktur Folder Repositori (Milestone 1):**

```
e-kos/
│
├── backend/                  # Express API (prefix /api)
│   ├── prisma/
│   │   ├── schema.prisma     # Definisi model dan relasi
│   │   ├── migrations/
│   │   └── seed.js           # Data awal (admin, tipe kamar, kamar)
│   ├── src/
│   │   ├── routes/           # Definisi routing per modul
│   │   ├── controllers/      # Handler request/response
│   │   ├── services/         # Logika bisnis (identitas, kamar, audit)
│   │   ├── middlewares/      # auth JWT, role guard, validasi, upload
│   │   ├── utils/            # helper JWT, hashing, response
│   │   └── app.js
│   ├── uploads/              # Dokumen identitas (privat)
│   ├── .env.example
│   └── server.js
│
├── frontend/                 # React + Vite (UI)
│   └── src/
│       ├── pages/            # public, auth, admin, calon-penyewa
│       ├── components/       # Komponen UI reusable
│       ├── layouts/          # PublicLayout, AdminLayout
│       ├── context/          # AuthContext
│       ├── routes/           # ProtectedRoute berbasis role
│       └── lib/              # API client (axios/fetch)
│
├── docs/                     # ERD dan dokumentasi tambahan
└── README.md                 # Dokumentasi utama proyek
```
