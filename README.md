# Travel Backend

Nama Kelompok:
1. Muhammad Rafi Febrio
2. Bintang Adhi susanto
3. Achmad Rafi Ilmal
4. Netha Yuwira

REST API + database untuk aplikasi Flutter pemesanan travel (Malang–Surabaya, dll).
**Self-hosted** — hanya Node.js + Express + SQLite. Tidak memakai Supabase, Firebase, atau BaaS lain.

Butuh **Node.js >= 22.13**. SQLite memakai modul bawaan Node (`node:sqlite`), jadi `npm install` tidak
mengompilasi apa pun dan tidak perlu Visual Studio/Python. Saat server start akan muncul peringatan
`ExperimentalWarning: SQLite is an experimental feature` — itu normal dan tidak berbahaya.

## Menjalankan

**Windows (Command Prompt)** — buka cmd di dalam folder project ini, lalu ketik satu per satu:

```bat
npm install
copy .env.example .env
npm run migrate
npm run seed
npm start
```

Atau cukup dua perintah (otomatis membuat `.env` + `JWT_SECRET` acak, migrate, dan seed):

```bat
npm install
npm run setup
npm start
```

Atau klik dua kali `setup.bat`.

**macOS / Linux**: sama, tetapi ganti `copy` dengan `cp`.

```bash
npm test                    # smoke test end-to-end (DB sementara)
```

Database tersimpan di `data/travel.db` (satu file, otomatis dibuat).

## Migration

Skema database dikelola lewat file SQL berurutan di folder `migrations/` (mis. `001_init.sql`).
`npm run migrate` menerapkan migration yang belum dijalankan; riwayatnya dicatat di tabel
`schema_migrations`. Server juga menjalankannya otomatis saat start. Untuk mengubah skema,
tambahkan file baru (`002_nama_perubahan.sql`) — jangan mengedit migration yang sudah dipakai.

## Skema database

| Tabel | Isi |
|---|---|
| `users` | akun (email/username unik, password di-hash bcrypt) |
| `cities` | kota + titik jemput (dari `_cityOptions` di home_Page) |
| `vehicles` | kendaraan + jumlah kursi (dari `VehicleOption`) |
| `schedules` | jadwal tetap: rute + kendaraan + jam + harga |
| `trips` | jadwal pada tanggal tertentu (dibuat otomatis saat dicari) |
| `bookings` | pesanan: kode booking, invoice, status, total |
| `booking_seats` | kursi terpakai; `UNIQUE(trip_id, seat_label)` mencegah double-booking |
| `payments` | catatan pembayaran |

Status pesanan: `pending` → `paid` / `cancelled` / `expired`. Kursi dikunci saat `pending`
(default 15 menit, `BOOKING_HOLD_MINUTES`), lalu otomatis dilepas bila tidak dibayar.

## Endpoint

Semua respons JSON. Error selalu berbentuk `{"error":{"code":"...","message":"..."}}`.
Endpoint bertanda 🔒 butuh header `Authorization: Bearer <token>`.

| Method | Path | Fungsi | Halaman Flutter |
|---|---|---|---|
| POST | `/api/auth/register` | `{name,email,password,confirm_password?,username?,phone?}` → `{token,user}` | RegisterPage |
| POST | `/api/auth/login` | `{identifier,password}` (email **atau** username) → `{token,user}` | LoginPage |
| GET | `/api/auth/me` 🔒 | data user login | — |
| PATCH | `/api/users/me` 🔒 | ubah `name`, `phone` | AkunSayaPage |
| POST | `/api/users/me/password` 🔒 | `{current_password,new_password}` | AkunSayaPage |
| GET | `/api/cities` | daftar kota | HomePage (city picker) |
| GET | `/api/vehicles` | daftar kendaraan | — |
| GET | `/api/trips?origin=MLG&destination=SBY&date=2026-10-10&passengers=2` | cari jadwal + sisa kursi | VehicleSelectionPage |
| GET | `/api/trips/:id` | detail satu trip | — |
| GET | `/api/trips/:id/seats` | peta kursi `tersedia`/`terisi` | PilihKursiPage |
| POST | `/api/bookings` 🔒 | `{trip_id,seats:["3","4"]}` → kunci kursi, status `pending` | tombol di PilihKursiPage |
| POST | `/api/bookings/:id/pay` 🔒 | `{method:"qris"\|"ovo"\|"bank"}` → `paid` | tombol BAYAR |
| POST | `/api/bookings/:id/cancel` 🔒 | batalkan pesanan `pending` | — |
| GET | `/api/bookings?status=paid` 🔒 | riwayat pesanan | TiketSayaPage |
| GET | `/api/bookings/:id` 🔒 | detail/struk | PaymentSuccessPage |
| GET | `/health` | cek server hidup | — |



