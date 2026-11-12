# Dokumen Perencanaan & Desain Sistem (E-Kos)

**Mata Kuliah:** Pemrograman Web Dinamis

**Nama Kelompok:** Ngawiters

**Topik Proyek:** E-Kos (Sistem Informasi Penyewaan dan Pengelolaan Kos)

## 1. Identitas Kelompok

- **Ketua:** Muchsin Hidayat Julianto - 2400018234 (Back End + Database)

- **Anggota 1:** Dimas Pratama Anugraha - 2400018215 (Back End + Database)

- **Anggota 2:** Nurrahmat Eko Nugroho - 2400018200 (Front End + Desain UI/UX)

- **Anggota 3:** Rafi Satya Prayoga - 2400018208 (Front End + Desain UI/UX)

## 2. Rancangan Aplikasi Web

**Judul Aplikasi:** E-Kos

**Deskripsi Singkat:** E-Kos adalah aplikasi web untuk mengelola penyewaan dan operasional satu lokasi kos milik satu pemilik, yang memisahkan arsitektur _backend_ (REST API) dan _frontend_. Aplikasi dilengkapi autentikasi JWT dengan dua peran akun, yaitu Admin dan Penyewa; calon penyewa adalah akun terdaftar yang belum memiliki peran Penyewa. Calon penyewa mengisi data dan mengunggah dokumen identitas untuk diverifikasi admin, memilih kamar yang tersedia, menyetujui peraturan kos, lalu melakukan simulasi pembayaran yang diverifikasi admin. Setelah pembayaran disetujui, sewa menjadi aktif, peran Penyewa diberikan, dan bukti transaksi diterbitkan otomatis. Penyewa dapat memantau masa sewa, tagihan layanan tambahan, refund, pengaduan, dan perpanjangan, sedangkan admin memantau seluruh aktivitas melalui dashboard, laporan, dan catatan audit. Fokus pada tahap ini adalah perencanaan dan desain basis data, dengan rancangan struktur yang siap dikembangkan untuk modul operasional penyewa pada tahap berikutnya.

**Stack Teknologi:**

- **Frontend:** React + JavaScript, CSS, Vite.

- **Backend:** Node.js, Express.js (REST API), autentikasi JWT.

- **Basis Data:** PostgreSQL dengan Prisma ORM.

## 3. Hak Akses Pengguna

Sistem kami memiliki dua peran akun, yaitu Admin dan Penyewa. Calon penyewa adalah akun terdaftar yang belum memiliki peran Penyewa, sehingga pada basis data kami bedakan menjadi tiga nilai `role`.

| Fitur (Milestone 1)                          | Pengunjung | Calon Penyewa | Penyewa                  | Admin     |
| -------------------------------------------- | ---------- | ------------- | ------------------------ | --------- |
| Melihat katalog kamar, harga, dan peraturan  | Ya         | Ya            | Ya                       | Ya        |
| Registrasi dan login                         | Ya         | Ya            | Ya                       | Ya        |
| Mengisi data identitas dan unggah dokumen    | Tidak      | Ya            | Hanya jika diminta admin | Memeriksa |
| Verifikasi identitas                         | Tidak      | Tidak         | Tidak                    | Ya        |
| CRUD tipe kamar, kamar, harga, dan peraturan | Tidak      | Tidak         | Tidak                    | Ya        |
| Mengelola pengguna                           | Tidak      | Tidak         | Tidak                    | Ya        |
| Melihat riwayat aktivitas (audit log)        | Tidak      | Tidak         | Tidak                    | Ya        |

Seluruh pemeriksaan hak akses kami lakukan di sisi server (middleware Express), bukan hanya di tampilan.

## 4. Desain Basis Data (ERD)

Sistem menggunakan basis data relasional PostgreSQL yang dikelola melalui Prisma. Berikut adalah rancangan tabel utama untuk memenuhi fitur autentikasi, verifikasi identitas, pengelolaan kamar, penyewaan, dan pembayaran (tahap 1), serta persiapan modul operasional penyewa (tahap selanjutnya):

**Skema Relasi Dasar:**

- **Users** (1) ---- (0..1) **Tenant_Profiles** _(Satu akun memiliki satu profil penyewa)_

- **Users** (1) ---- m ----> (M) **Identity_Documents** _(Satu akun memiliki riwayat unggahan dokumen identitas)_

- **Room_Types** (1) ---- m ----> (M) **Rooms** _(Satu tipe kamar menaungi banyak kamar)_

- **Room_Types** (1) ---- m ----> (M) **Rental_Prices** _(Satu tipe kamar memiliki harga untuk tiap periode sewa)_

- **Users** (1) ---- m ----> (M) **Payment_Transactions** _(Satu akun melakukan banyak percobaan pembayaran)_

- **Rooms** (1) ---- m ----> (M) **Payment_Transactions** _(Satu kamar dapat menjadi objek banyak transaksi sewa)_

- **Payment_Transactions** (1) ---- (0..1) **Leases** _(Satu transaksi sewa awal yang disetujui menghasilkan satu masa sewa)_

- **Payment_Transactions** (1) ---- (0..1) **Transaction_Receipts** _(Satu transaksi memiliki satu bukti transaksi)_

- **Leases** (1) ---- m ----> (M) **Lease_Renewals** _(Satu masa sewa dapat diperpanjang beberapa kali)_

- **Payment_Transactions** (1) ---- m ----> (M) **Refund_Requests** _(Satu transaksi menjadi dasar pengajuan refund)_

- **House_Rules** (1) ---- m ----> (M) **Rule_Acknowledgments** _(Satu versi peraturan disetujui oleh banyak pengguna)_

- **Additional_Services** (1) ---- m ----> (M) **Service_Bills** _(Satu layanan tambahan ditagihkan berulang)_

- **Leases** (1) ---- m ----> (M) **Service_Bills** _(Satu masa sewa memiliki banyak tagihan layanan)_

- **Service_Bills** (1) ---- m ----> (M) **Payment_Transactions** _(Satu tagihan dapat memiliki beberapa percobaan pembayaran)_

- **Users** (1) ---- m ----> (M) **Complaints** _(Satu penyewa mengajukan banyak pengaduan)_

- **Complaints** (1) ---- m ----> (M) **Complaint_Notes** _(Satu pengaduan memiliki banyak catatan tindak lanjut)_

- **Users** (1) ---- m ----> (M) **Notifications** _(Satu akun menerima banyak pengingat)_

- **Users** (1) ---- m ----> (M) **Audit_Logs** _(Satu akun memiliki banyak riwayat aktivitas)_

**Diagram ERD:**

```
erDiagram
    USERS ||--o| TENANT_PROFILES : "memiliki"
    USERS ||--o{ IDENTITY_DOCUMENTS : "mengunggah"
    USERS ||--o{ PAYMENT_TRANSACTIONS : "melakukan"
    USERS ||--o{ LEASES : "menyewa"
    USERS ||--o{ REFUND_REQUESTS : "mengajukan"
    USERS ||--o{ SERVICE_BILLS : "ditagih"
    USERS ||--o{ COMPLAINTS : "membuat"
    USERS ||--o{ RULE_ACKNOWLEDGMENTS : "menyetujui"
    USERS ||--o{ NOTIFICATIONS : "menerima"
    USERS ||--o{ AUDIT_LOGS : "pelaku aksi"

    ROOM_TYPES ||--o{ ROOMS : "mengelompokkan"
    ROOM_TYPES ||--o{ RENTAL_PRICES : "memiliki harga"
    ROOMS ||--o{ LEASES : "disewa melalui"
    ROOMS ||--o{ PAYMENT_TRANSACTIONS : "dipilih pada"

    PAYMENT_TRANSACTIONS ||--o| LEASES : "mengaktifkan sewa awal"
    LEASES ||--o{ LEASE_RENEWALS : "diperpanjang"
    PAYMENT_TRANSACTIONS ||--o| LEASE_RENEWALS : "membayar perpanjangan"
    PAYMENT_TRANSACTIONS ||--o| TRANSACTION_RECEIPTS : "menghasilkan"
    PAYMENT_TRANSACTIONS ||--o{ REFUND_REQUESTS : "menjadi dasar"

    HOUSE_RULES ||--o{ RULE_ACKNOWLEDGMENTS : "disetujui pada"
    ADDITIONAL_SERVICES ||--o{ SERVICE_BILLS : "ditagihkan"
    LEASES ||--o{ SERVICE_BILLS : "terkait"
    SERVICE_BILLS ||--o{ PAYMENT_TRANSACTIONS : "dibayar melalui"

    COMPLAINTS ||--o{ COMPLAINT_NOTES : "memiliki catatan"

    KOS_PROFILE
    DISCOUNTS

```

_Catatan: tabel `discounts` tidak memiliki garis relasi karena diskon berlaku berdasarkan periode sewa, dan tabel `kos_profile` hanya berisi satu baris karena sistem mengelola satu lokasi kos._

**Struktur Tabel Awal:**

1.  **Tabel `users`** (Modul Autentikasi)
    - `id` (UUID, Primary Key)

    - `name` (VARCHAR)

    - `email` (VARCHAR, Unique)

    - `password_hash` (VARCHAR)

    - `role` (ENUM: 'ADMIN', 'PROSPECTIVE_TENANT', 'TENANT')

    - `is_active` (BOOLEAN, default: true)

    - `token_version` (INT, default: 1 - _untuk validasi sesi JWT_)

    - `deleted_at` (TIMESTAMP, _Soft Delete_)

    - `created_at`, `updated_at` (TIMESTAMP)

2.  **Tabel `tenant_profiles`** (Modul Pengguna)
    - `id` (UUID, Primary Key)

    - `user_id` (UUID, Foreign Key references `users.id`, Unique)

    - `full_name` (VARCHAR)

    - `phone` (VARCHAR)

    - `identity_number` (VARCHAR)

    - `address` (VARCHAR)

    - `identity_status` (ENUM: 'NOT_UPLOADED', 'PENDING_VERIFICATION', 'NEEDS_REPAIR', 'APPROVED', 'REJECTED_FINAL')

    - `updated_at` (TIMESTAMP)

3.  **Tabel `identity_documents`** (Modul Verifikasi Identitas)
    - `id` (UUID, Primary Key)

    - `user_id` (UUID, Foreign Key references `users.id`)

    - `reviewed_by` (UUID, Foreign Key references `users.id`, nullable - _admin pemeriksa_)

    - `document_type` (VARCHAR)

    - `file_path` (VARCHAR - _lokasi berkas di penyimpanan privat_)

    - `attempt_no` (INT - _urutan unggahan/riwayat unggah ulang_)

    - `status` (ENUM: 'PENDING_VERIFICATION', 'NEEDS_REPAIR', 'APPROVED', 'REJECTED_FINAL')

    - `rejection_reason`, `repair_instruction` (TEXT)

    - `submitted_at`, `reviewed_at` (TIMESTAMP)

4.  **Tabel `kos_profile`** (Modul Informasi Kos)
    - `id` (INT, Primary Key - _satu baris_)

    - `name`, `address`, `description`, `contact_phone` (VARCHAR/TEXT)

    - `photos` (JSON)

    - `updated_at` (TIMESTAMP)

5.  **Tabel `room_types`** (Modul Kamar)
    - `id` (UUID, Primary Key)

    - `name` (VARCHAR, Unique)

    - `description` (TEXT)

    - `facilities` (JSON)

    - `photos` (JSON)

    - `created_at`, `updated_at` (TIMESTAMP)

6.  **Tabel `rooms`** (Modul Kamar)
    - `id` (UUID, Primary Key)

    - `room_number` (VARCHAR, Unique)

    - `room_type_id` (UUID, Foreign Key references `room_types.id`)

    - `floor` (INT)

    - `status` (ENUM: 'AVAILABLE', 'LOCKED', 'OCCUPIED', 'MAINTENANCE')

    - `notes` (TEXT)

    - `updated_at` (TIMESTAMP)

7.  **Tabel `rental_prices`** (Modul Harga)
    - `id` (UUID, Primary Key)

    - `room_type_id` (UUID, Foreign Key references `room_types.id`)

    - `period` (ENUM: 'MONTHLY', 'SIX_MONTHS', 'ONE_YEAR')

    - `price` (DECIMAL)

    - `effective_from` (DATE)

    - `is_active` (BOOLEAN)

8.  **Tabel `discounts`** (Modul Harga)
    - `id` (UUID, Primary Key)

    - `period` (ENUM: 'MONTHLY', 'SIX_MONTHS', 'ONE_YEAR')

    - `discount_type` (ENUM: 'PERCENT', 'FIXED')

    - `value` (DECIMAL)

    - `start_date`, `end_date` (DATE)

    - `is_active` (BOOLEAN)

9.  **Tabel `house_rules`** (Modul Peraturan Kos)
    - `id` (UUID, Primary Key)

    - `version` (INT, Unique)

    - `title` (VARCHAR)

    - `content` (TEXT)

    - `document_path` (VARCHAR - _merujuk `house-rules.pdf`_)

    - `is_active` (BOOLEAN)

    - `published_at` (TIMESTAMP)

10. **Tabel `rule_acknowledgments`** (Modul Peraturan Kos)
    - `id` (UUID, Primary Key)

    - `user_id` (UUID, Foreign Key references `users.id`)

    - `house_rule_id` (UUID, Foreign Key references `house_rules.id`)

    - `payment_transaction_id` (UUID, Foreign Key references `payment_transactions.id`)

    - `acknowledged_at` (TIMESTAMP)

11. **Tabel `payment_transactions`** (Modul Pembayaran)
    - `id` (UUID, Primary Key)

    - `transaction_code` (VARCHAR, Unique)

    - `user_id` (UUID, Foreign Key references `users.id`)

    - `type` (ENUM: 'INITIAL_RENT', 'RENEWAL', 'SERVICE_BILL')

    - `room_id` (UUID, Foreign Key references `rooms.id`, nullable - _untuk sewa awal_)

    - `lease_id` (UUID, Foreign Key references `leases.id`, nullable - _untuk perpanjangan_)

    - `service_bill_id` (UUID, Foreign Key references `service_bills.id`, nullable - _untuk tagihan layanan_)

    - `period` (ENUM, nullable)

    - `amount` (DECIMAL)

    - `payment_method` (ENUM: 'SIMULATION')

    - `is_simulation` (BOOLEAN, default: true)

    - `status` (ENUM: 'PENDING_VERIFICATION', 'APPROVED', 'REJECTED')

    - `rejection_reason` (TEXT)

    - `verified_by` (UUID, Foreign Key references `users.id`, nullable)

    - `verified_at`, `created_at` (TIMESTAMP)

12. **Tabel `leases`** (Modul Masa Sewa)
    - `id` (UUID, Primary Key)

    - `user_id` (UUID, Foreign Key references `users.id`)

    - `room_id` (UUID, Foreign Key references `rooms.id`)

    - `initial_payment_id` (UUID, Foreign Key references `payment_transactions.id`, Unique)

    - `period` (ENUM: 'MONTHLY', 'SIX_MONTHS', 'ONE_YEAR')

    - `start_date`, `end_date` (DATE)

    - `status` (ENUM: 'ACTIVE', 'ENDED', 'CANCELLED')

    - `created_at`, `updated_at` (TIMESTAMP)

13. **Tabel `transaction_receipts`** (Modul Bukti Transaksi)
    - `id` (UUID, Primary Key)

    - `payment_transaction_id` (UUID, Foreign Key references `payment_transactions.id`, Unique)

    - `receipt_number` (VARCHAR, Unique)

    - `is_simulation` (BOOLEAN)

    - `issued_at` (TIMESTAMP)

14. **Tabel `audit_logs`** (Modul Audit)
    - `id` (BIGINT, Primary Key)

    - `actor_id` (UUID, Foreign Key references `users.id`)

    - `action` (VARCHAR) _(Contoh: 'identity.approve', 'payment.reject', 'refund.process')_

    - `entity_type` (VARCHAR), `entity_id` (UUID)

    - `old_value`, `new_value` (JSON)

    - `reason` (TEXT)

    - `ip_address` (VARCHAR)

    - `created_at` (TIMESTAMP)

15. **Tabel `lease_renewals`** (Persiapan Tahap Berikutnya)
    - `id` (UUID, Primary Key)

    - `lease_id` (UUID, Foreign Key references `leases.id`)

    - `payment_transaction_id` (UUID, Foreign Key references `payment_transactions.id`, Unique)

    - `period` (ENUM), `months_added` (INT)

    - `previous_end_date`, `new_end_date` (DATE)

    - `status` (ENUM: 'APPLIED', 'CANCELLED')

    - `created_at` (TIMESTAMP)

16. **Tabel `refund_requests`** (Persiapan Tahap Berikutnya)
    - `id` (UUID, Primary Key)

    - `payment_transaction_id` (UUID, Foreign Key references `payment_transactions.id`)

    - `requested_by` (UUID, Foreign Key references `users.id`)

    - `reason` (TEXT)

    - `destination_type` (ENUM: 'BANK', 'E_WALLET')

    - `destination_name`, `account_holder_name` (VARCHAR)

    - `destination_number` (VARCHAR - _dienkripsi_)

    - `amount` (DECIMAL)

    - `status` (ENUM: 'PENDING_REVIEW', 'IN_PROGRESS', 'APPROVED', 'REJECTED', 'COMPLETED')

    - `decision_reason` (TEXT)

    - `processed_by` (UUID, Foreign Key references `users.id`, nullable)

    - `requested_at`, `due_at`, `processed_at` (TIMESTAMP)

17. **Tabel `additional_services`** (Persiapan Tahap Berikutnya)
    - `id` (UUID, Primary Key)

    - `name` (VARCHAR, Unique)

    - `description` (TEXT)

    - `default_rate` (DECIMAL)

    - `billing_cycle` (VARCHAR)

    - `is_mandatory`, `is_active` (BOOLEAN)

18. **Tabel `service_bills`** (Persiapan Tahap Berikutnya)
    - `id` (UUID, Primary Key)

    - `user_id` (UUID, Foreign Key references `users.id`)

    - `lease_id` (UUID, Foreign Key references `leases.id`)

    - `additional_service_id` (UUID, Foreign Key references `additional_services.id`)

    - `period_start`, `period_end`, `due_date` (DATE)

    - `amount` (DECIMAL)

    - `status` (ENUM: 'UNPAID', 'PENDING_VERIFICATION', 'PAID', 'OVERDUE', 'CANCELLED')

    - `created_by` (UUID, Foreign Key references `users.id`)

    - `created_at` (TIMESTAMP)

19. **Tabel `complaints`** (Persiapan Tahap Berikutnya)
    - `id` (UUID, Primary Key)

    - `user_id` (UUID, Foreign Key references `users.id`)

    - `title` (VARCHAR)

    - `description` (TEXT)

    - `status` (ENUM: 'SUBMITTED', 'IN_PROGRESS', 'RESOLVED')

    - `created_at`, `resolved_at` (TIMESTAMP)

20. **Tabel `complaint_notes`** (Persiapan Tahap Berikutnya)
    - `id` (UUID, Primary Key)

    - `complaint_id` (UUID, Foreign Key references `complaints.id`)

    - `author_id` (UUID, Foreign Key references `users.id`)

    - `note` (TEXT)

    - `is_visible_to_tenant` (BOOLEAN)

    - `created_at` (TIMESTAMP)

21. **Tabel `notifications`** (Persiapan Tahap Berikutnya)
    - `id` (UUID, Primary Key)

    - `user_id` (UUID, Foreign Key references `users.id`, nullable - _kosong untuk pengingat admin_)

    - `target_role` (VARCHAR)

    - `type` (ENUM: 'LEASE_ENDING', 'BILL_UNPAID', 'BILL_OVERDUE')

    - `title` (VARCHAR), `message` (TEXT)

    - `reference_type` (VARCHAR), `reference_id` (UUID)

    - `is_read` (BOOLEAN, default: false)

    - `created_at` (TIMESTAMP)

## 5. Repositori & Inisialisasi Proyek

**Tautan Repositori GitHub:** https://github.com/nurrahmat-eko/E-Kos

Pastikan saya diinvite untuk akses repo (github/gitlab: supriyanto@uad.ac.id)

**Status Setup Lingkungan:**

- Repositori menggunakan format _Monorepo_ dengan folder `backend` dan `frontend`.

- _Backend_ menggunakan Node.js + Express.js, terhubung ke PostgreSQL melalui Prisma ORM.

- _Frontend_ diinisialisasi menggunakan React dengan _build tool_ Vite dan integrasi CSS.

**Struktur Folder Repositori (Milestone 1):**

```
e-kos/
│
├── backend/                                # Node.js + Express.js (REST API, prefix /api)
│   ├── prisma/                             # Skema dan migrasi basis data (Prisma)
│   │   ├── migrations/
│   │   ├── schema.prisma
│   │   └── seed.js
│   │
│   ├── scripts/                            # Skrip bantu pengembangan
│   │
│   ├── src/
│   │   │
│   │   ├── config/                         # Konfigurasi env dan Prisma Client
│   │   │   ├── env.js
│   │   │   └── prisma.js
│   │   │
│   │   ├── security/                       # JWT dan hashing password
│   │   │   ├── jwt.js
│   │   │   └── password.js
│   │   │
│   │   ├── middlewares/                    # Autentikasi, otorisasi, validasi, upload, error
│   │   │   ├── authenticate.js
│   │   │   ├── authorize.js
│   │   │   ├── validateRequest.js
│   │   │   ├── upload.js
│   │   │   ├── errorHandler.js
│   │   │   └── notFound.js
│   │   │
│   │   ├── routes/                         # Penggabung seluruh rute (prefix /api)
│   │   │   └── index.js
│   │   │
│   │   ├── modules/                        # Modul per domain (routes, controller, service, repository, validation)
│   │   │   │
│   │   │   ├── auth/
│   │   │   │   ├── auth.routes.js
│   │   │   │   ├── auth.controller.js
│   │   │   │   ├── auth.service.js
│   │   │   │   ├── auth.repository.js
│   │   │   │   └── auth.validation.js
│   │   │   │
│   │   │   ├── users/
│   │   │   ├── kos/
│   │   │   ├── rooms/
│   │   │   ├── pricing/
│   │   │   ├── identity/
│   │   │   ├── leases/
│   │   │   ├── payments/
│   │   │   ├── refunds/
│   │   │   ├── receipts/
│   │   │   ├── additional-services/
│   │   │   ├── service-bills/
│   │   │   ├── complaints/
│   │   │   ├── notifications/
│   │   │   ├── dashboard/
│   │   │   ├── reports/
│   │   │   ├── audit-logs/
│   │   │   └── rule-acknowledgments/
│   │   │
│   │   ├── integrations/                   # Integrasi penyimpanan file privat
│   │   │   └── storage/
│   │   │       └── privateFileStorage.js
│   │   │
│   │   ├── jobs/                           # Tugas terjadwal (masa sewa dan pengingat)
│   │   │   ├── expireLeases.job.js
│   │   │   └── generateReminders.job.js
│   │   │
│   │   ├── utils/                          # Fungsi bantu
│   │   │   ├── ApiError.js
│   │   │   ├── asyncHandler.js
│   │   │   ├── date.js
│   │   │   └── currency.js
│   │   │
│   │   ├── tests/                          # Pengujian (unit, integration, e2e)
│   │   │   ├── unit/
│   │   │   ├── integration/
│   │   │   └── e2e/
│   │   │
│   │   ├── app.js
│   │   └── server.js
│   │
│   ├── .env
│   ├── .env.example
│   ├── .env.test
│   ├── .gitignore
│   ├── package.json
│   └── package-lock.json
│
└── frontend/                               # React + Vite (UI)
    ├── public/                             # Aset statis dan dokumen peraturan kos
    │   ├── images/
    │   └── documents/
    │       └── house-rules.pdf
    │
    ├── src/
    │   │
    │   ├── assets/                         # Gambar, ikon, font
    │   │   ├── images/
    │   │   ├── icons/
    │   │   └── fonts/
    │   │
    │   ├── components/                     # Komponen UI reusable
    │   │   ├── ui/
    │   │   ├── layout/
    │   │   └── feedback/
    │   │
    │   ├── features/                       # Logika per fitur (sejajar dengan modul backend)
    │   │   ├── auth/
    │   │   ├── users/
    │   │   ├── kos/
    │   │   ├── rooms/
    │   │   ├── pricing/
    │   │   ├── identity/
    │   │   ├── leases/
    │   │   ├── payments/
    │   │   ├── refunds/
    │   │   ├── receipts/
    │   │   ├── additional-services/
    │   │   ├── service-bills/
    │   │   ├── complaints/
    │   │   ├── notifications/
    │   │   ├── dashboard/
    │   │   ├── reports/
    │   │   └── audit-logs/
    │   │
    │   ├── pages/                          # Halaman per peran pengguna
    │   │   ├── public/
    │   │   ├── auth/
    │   │   ├── prospective-tenant/
    │   │   ├── tenant/
    │   │   └── admin/
    │   │
    │   ├── routes/                         # Definisi rute dan proteksi peran
    │   │   ├── index.jsx
    │   │   ├── ProtectedRoute.jsx
    │   │   ├── RoleRoute.jsx
    │   │   └── routePaths.js
    │   │
    │   ├── services/                       # Klien API
    │   │   └── apiClient.js
    │   │
    │   ├── context/                        # State autentikasi
    │   │   └── AuthContext.jsx
    │   │
    │   ├── hooks/                          # Custom hooks
    │   │
    │   ├── constants/                      # Konstanta (status, peran)
    │   │
    │   ├── utils/                          # Fungsi bantu
    │   │
    │   ├── styles/                         # Styling CSS
    │   │
    │   │
    │   ├── tests/                          # Pengujian frontend
    │   │
    │   ├── App.jsx
    │   └── main.jsx
    │
    ├── .env
    ├── .env.example
    ├── .gitignore
    ├── index.html
    ├── package.json
    ├── package-lock.json
    ├── postcss.config.js
    └── vite.config.js

```

## 6. Rencana Pengerjaan dan Pembagian Tugas

| Anggota             | Tanggung Jawab Milestone 1                                                                                              |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Muchsin (BE, Ketua) | Setup monorepo, autentikasi JWT dan middleware role, modul verifikasi identitas, audit log, koordinasi repositori       |
| Dimas (BE)          | Skema Prisma dan migrasi, seeder, CRUD tipe kamar, kamar, harga, dan peraturan, upload file                             |
| Nurrahmat (FE)      | Setup Vite dan css styling, layout dan routing, halaman login dan registrasi, dashboard admin, halaman kelola kamar     |
| Rafi (FE)           | Katalog kamar publik dan detail kamar, formulir identitas dan unggah dokumen, halaman status identitas, kelola pengguna |
