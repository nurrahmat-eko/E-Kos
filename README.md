# E-Kos

**Cari Kos, Pesan Mudah, Tinggal Nyaman.**

E-Kos adalah aplikasi web untuk pengelolaan satu lokasi kos milik satu pemilik. Arsitekturnya memisahkan *backend* (REST API) dan *frontend* (SPA). Proyek ini dikerjakan untuk mata kuliah **Pemrograman Web Dinamis 2026** oleh kelompok **Ngawiters**.

> **Status:** Milestone 1 (fondasi sistem). Dokumen ini hanya menjelaskan fitur dan cara menjalankan proyek pada tahap Milestone 1.

---

## Tim

| Peran | Nama | NIM | Posisi |
| --- | --- | --- | --- |
| Ketua | Muchsin Hidayat Julianto | 2400018234 | Backend Developer |
| Anggota | Dimas Pratama Anugraha | 2400018215 | Backend Developer |
| Anggota | Nurrahmat Eko Nugroho | 2400018200 | Frontend Developer |
| Anggota | Rafi Satya Prayoga | 2400018208 | Frontend Developer |

---

## Fitur Milestone 1

- **Autentikasi dan hak akses:** registrasi, login, dan pembagian role `admin`, `calon_penyewa`, dan `penyewa` (JWT, password di-hash dengan bcrypt).
- **Katalog publik:** pengunjung dapat melihat daftar kamar, detail kamar, harga per periode sewa, dan peraturan kos aktif.
- **Pengelolaan data master (admin):** CRUD tipe kamar, kamar, harga sewa (bulanan, 6 bulan, 1 tahun), dan peraturan kos.
- **Verifikasi identitas:** calon penyewa mengisi data dan mengunggah dokumen identitas; admin menyetujui, meminta perbaikan, atau menolak final dengan alasan. Riwayat unggahan tersimpan.
- **Pengelolaan pengguna (admin).**
- **Audit log:** tindakan penting (login, verifikasi identitas, perubahan harga dan status kamar) tercatat beserta pelaku dan waktunya.

---

## Stack Teknologi

| Lapisan | Teknologi |
| --- | --- |
| Frontend | React (JavaScript), Vite, Tailwind CSS |
| Backend | Node.js, Express.js, JWT, bcrypt |
| Basis data | PostgreSQL dengan Prisma ORM |

---

## Struktur Folder

```
e-kos/
├── backend/                  # Express API (prefix /api)
│   ├── prisma/
│   │   ├── schema.prisma     # Model dan relasi
│   │   ├── migrations/
│   │   └── seed.js           # Data awal (admin, tipe kamar, kamar)
│   ├── src/
│   │   ├── routes/           # Routing per modul
│   │   ├── controllers/      # Handler request/response
│   │   ├── services/         # Logika bisnis
│   │   ├── middlewares/      # Auth JWT, role guard, validasi, upload
│   │   ├── utils/            # Helper JWT, hashing, response
│   │   └── app.js
│   ├── uploads/              # Dokumen identitas (privat)
│   ├── .env.example
│   └── server.js
│
├── frontend/                 # React + Vite
│   └── src/
│       ├── pages/            # public, auth, admin, calon-penyewa
│       ├── components/       # Komponen UI reusable
│       ├── layouts/          # PublicLayout, AdminLayout
│       ├── context/          # AuthContext
│       ├── routes/           # ProtectedRoute berbasis role
│       └── lib/              # API client
│
├── docs/                     # ERD dan dokumentasi tambahan
└── README.md
```

---

## Prasyarat

- Node.js 20 atau lebih baru
- npm
- PostgreSQL (lokal atau layanan cloud)
- Git

---

## Cara Menjalankan

### 1. Clone repositori

```bash
git clone https://github.com/nurrahmat-eko/E-Kos.git
cd E-Kos
```

### 2. Jalankan backend

```bash
cd backend
npm install
cp .env.example .env
```

Isi file `.env`:

```env
PORT=5000
DATABASE_URL="postgresql://USER:PASSWORD@localhost:5432/ekos"
JWT_SECRET="isi-dengan-string-acak-yang-panjang"
JWT_EXPIRES_IN="1d"
CLIENT_URL="http://localhost:5173"

# Akun admin awal untuk seeder
ADMIN_NAME="Admin E-Kos"
ADMIN_EMAIL="admin@ekos.test"
ADMIN_PASSWORD="ganti-password-ini"
```

Jalankan migrasi, seeder, lalu server:

```bash
npx prisma migrate dev
npx prisma db seed
npm run dev
```

API berjalan di `http://localhost:5000/api`.

### 3. Jalankan frontend

Buka terminal baru:

```bash
cd frontend
npm install
cp .env.example .env
npm run dev
```

Isi `frontend/.env`:

```env
VITE_API_URL=http://localhost:5000/api
```

Aplikasi berjalan di `http://localhost:5173`.

---

## Role dan Hak Akses

| Fitur | Pengunjung | Calon Penyewa | Penyewa | Admin |
| --- | --- | --- | --- | --- |
| Melihat katalog kamar, harga, peraturan | Ya | Ya | Ya | Ya |
| Registrasi dan login | Ya | Ya | Ya | Ya |
| Mengisi data identitas dan unggah dokumen | Tidak | Ya | Hanya jika diminta admin | Memeriksa |
| Verifikasi identitas | Tidak | Tidak | Tidak | Ya |
| CRUD tipe kamar, kamar, harga, peraturan | Tidak | Tidak | Tidak | Ya |
| Mengelola pengguna | Tidak | Tidak | Tidak | Ya |
| Melihat audit log | Tidak | Tidak | Tidak | Ya |

Pemeriksaan hak akses dilakukan di sisi server melalui middleware, bukan hanya di tampilan.

---

## Ringkasan Endpoint API

| Modul | Endpoint | Akses |
| --- | --- | --- |
| Auth | `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me` | Publik / login |
| Katalog | `GET /api/rooms`, `GET /api/rooms/:id`, `GET /api/house-rules/active` | Publik |
| Tipe kamar | `GET/POST/PUT/DELETE /api/room-types` | Admin |
| Kamar | `POST/PUT/DELETE /api/rooms` | Admin |
| Harga sewa | `GET/POST/PUT/DELETE /api/rental-prices` | Admin |
| Peraturan kos | `GET/POST/PUT /api/house-rules` | Admin |
| Identitas (pengguna) | `GET/POST /api/identity`, `POST /api/identity/reupload` | Calon penyewa |
| Identitas (admin) | `GET /api/admin/identities`, `PATCH /api/admin/identities/:id/review` | Admin |
| Pengguna | `GET/PUT/DELETE /api/users` | Admin |
| Audit log | `GET /api/audit-logs` | Admin |

---

## Basis Data

Tabel pada Milestone 1:

| Tabel | Fungsi |
| --- | --- |
| `users` | Akun, role, dan status |
| `identity_documents` | Data identitas dan status verifikasi per akun |
| `identity_uploads` | Riwayat unggahan dokumen identitas |
| `room_types` | Tipe kamar dan fasilitas |
| `rooms` | Data kamar dan statusnya |
| `rental_prices` | Harga dan diskon per tipe kamar dan periode sewa |
| `house_rules` | Versi peraturan kos |
| `audit_logs` | Riwayat tindakan penting |

Relasi utama: satu tipe kamar memiliki banyak kamar dan banyak harga; satu pengguna memiliki satu data identitas dengan banyak riwayat unggahan; satu pengguna memiliki banyak audit log.

---

## Catatan Keamanan

- Jangan commit file `.env`. Gunakan `.env.example` sebagai acuan.
- Password disimpan sebagai hash bcrypt.
- Dokumen identitas disimpan di folder privat dan hanya dapat diakses lewat endpoint yang terotorisasi, tidak lewat folder publik.
- Pengguna hanya dapat melihat data miliknya sendiri; akses data pengguna lain dibatasi di sisi server.

---

## Pembagian Tugas Milestone 1

| Anggota | Tanggung Jawab |
| --- | --- |
| Muchsin | Setup monorepo, autentikasi JWT dan middleware role, verifikasi identitas, audit log |
| Dimas | Skema Prisma dan migrasi, seeder, CRUD tipe kamar, kamar, harga, dan peraturan, upload file |
| Nurrahmat | Setup Vite dan Tailwind, layout dan routing, halaman auth, dashboard admin, halaman kelola kamar |
| Rafi | Katalog kamar publik dan detail, formulir identitas dan unggah dokumen, halaman status identitas, kelola pengguna |

---

## Lisensi

Proyek ini dibuat untuk keperluan tugas mata kuliah.
