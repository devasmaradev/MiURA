# Aplikasi Guru AI — MiURA

Platform administrasi akademik guru: pengelolaan data siswa, kelas, absensi, penilaian, agenda mengajar, bimbingan guru wali, kartu pelajar QR, laporan PDF, dan generator perangkat ajar berbasis AI (Gemini).

## Teknologi

- React 19 + TypeScript + Vite + Tailwind CSS v4
- Express (server) + Firebase Firestore (realtime) + Firebase Auth (email/password)
- Gemini AI via `@google/genai` (server-side, `GEMINI_API_KEY`)
- jsPDF / jspdf-autotable (laporan), jsQR + qrcode (kartu & presensi), xlsx (import siswa), DOMPurify (sanitasi HTML AI)

## Menjalankan Lokal

**Prasyarat:** Node.js 20+ (package manager: npm)

1. Install dependensi:
   ```
   npm install
   ```
2. Siapkan konfigurasi:
   - Buat `.env.local` lalu isi `GEMINI_API_KEY=<key>` (endpoint AI). Server jalan tanpa key, hanya fitur AI yang akan mengembalikan error 500.
   - Isi `firebase-applet-config.json` dengan kredensial Firebase Web App (apiKey, authDomain, projectId, storageBucket, messagingSenderId, appId). Jika dikosongkan, aplikasi tetap dapat di-build/dibuka tetapi seluruh fitur data non-aktif dan menampilkan pesan "Database belum terhubung". Pastikan Firestore terprovisi dan rules ter-deploy: `firestore.rules`.
3. Jalankan:
   ```
   npm run dev
   ```
   Server berjalan di http://localhost:3000 (Vite middleware via tsx).

## Build & Produksi

```
npm run build   # bundle client (vite) + server (esbuild) ke dist/
npm start       # node dist/server.cjs — menyajikan dist/ + API
```

## Keamanan (penting sebelum deploy)

- Aplikasi menggunakan **email/password** (Firebase Auth). Aturannya:
  1. Firebase Console → Authentication → Sign-in method → aktifkan **Email/Password**.
  2. Firebase Console → Authentication → Users → **Add user** (buat akun guru/pengelola).
  3. Deploy `firestore.rules` (hanya `request.auth != null` yang diizinkan baca/tulis).
- Tanpa akun yang valid, aplikasi menampilkan halaman login dan **tidak** dapat mengakses data.
- **Batasan saat ini:** semua user yang terautentikasi memiliki akses penuh ke seluruh koleksi (satu-peran). Untuk multi-peran (mis. admin vs guru), bagi rules di `firestore.rules` berdasar klaim/custom claims (`ponytail`).
- Seluruh output HTML dari Gemini disanitasi dengan DOMPurify sebelum dirender/dicetak.
- Endpoint AI memiliki rate limiter in-memory (20 permintaan/menit/IP).

## Script

| Perintah | Fungsi |
| -------- | ------ |
| `npm run dev` | Jalankan server dev (tsx) |
| `npm run build` | Build client + server ke `dist/` |
| `npm start` | Jalankan hasil build |
| `npm run lint` | Type-check `tsc --noEmit` |
| `npm run clean` | Hapus `dist/` dan `server.js` |

## Struktur Utama

```
src/
  App.tsx                 # routing tab + state global + langganan Firestore
  components/             # satu komponen per menu
  lib/
    firebase.ts           # init Firebase, auth, helper CRUD & cascade
    sanitize.ts           # DOMPurify untuk HTML dari Gemini
    date.ts               # helper tanggal lokal (hindari toISOString UTC)
    swal.ts               # konfigurasi dialog/toast
server.ts                 # API Gemini + rate limit + statik produksi
firestore.rules           # rules Firestore (wajib di-deploy)
firebase-applet-config.json  # kredensial Firebase (diisi / inject saat deploy)
```