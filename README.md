# Misi Penjaga Norma — starter project

Aplikasi React + Vite untuk game Pendidikan Pancasila kelas VII. Termasuk 4 level dan 20 tantangan, umpan balik per soal, refleksi, halaman login guru demo, dashboard demo, detail jawaban, pencarian, filter, urut skor, ekspor CSV, dan skema awal Supabase.

## Status penting

**Ini adalah starter project, bukan website yang sudah dipublikasikan.** Saat `.env.local` belum diisi, aplikasi berjalan dalam mode demo dan menyimpan hasil hanya pada `localStorage` browser yang sama. Data demo tidak masuk ke database online dan tidak bisa terlihat dari perangkat guru lain.

Login demo (hanya untuk uji tampilan lokal):
- Username: `guru_demo`
- Password: `NormaDemo7!`

Jangan gunakan kredensial demo untuk data siswa nyata. Jangan bagikan website sebelum autentikasi produksi dan API siswa selesai dibuat dan diuji.

## 1. Jalankan lokal

Perlu Node.js LTS terpasang.

```bash
npm install
npm run dev
```

Buka alamat lokal yang ditampilkan Vite. Untuk uji build:

```bash
npm run build
npm run preview
```

## 2. Buat database Supabase

1. Buka https://supabase.com dan buat akun.
2. Buat project baru, pilih kata sandi database yang kuat dan simpan secara aman.
3. Buka **SQL Editor** di dashboard Supabase.
4. Salin isi `supabase/schema.sql`, jalankan, lalu periksa hasilnya.
5. Ambil Project URL dan publishable/anon key dari Project Settings / API.
6. Salin `.env.example` menjadi `.env.local`, lalu isi:

```env
VITE_SUPABASE_URL=https://PROJECT_ID.supabase.co
VITE_SUPABASE_ANON_KEY=KUNCI_PUBLISHABLE_ATAU_ANON
```

Kunci publishable/anon boleh digunakan di browser hanya bila RLS dan kebijakan akses sudah benar. **Jangan pernah memasukkan `service_role` key atau rahasia database ke `VITE_*` atau kode frontend.**

### Penting tentang database

Skema SQL menyiapkan struktur dan kebijakan baca guru. Namun, agar alur siswa memakai kode kelas dengan aman, perlu dibuat dan diuji Supabase Edge Functions untuk:
- `student-join`: validasi kode kelas aktif dan membuat identitas/percobaan siswa.
- `student-answer`: mengambil kunci jawaban di server, menyimpan satu jawaban per soal, dan menghitung poin di server.
- `student-finish`: menghitung skor final dari jawaban tersimpan, menyimpan refleksi/komitmen.
- `teacher-login`: login username + password tanpa mengekspos daftar akun.

UI starter ini belum memanggil fungsi tersebut. Mengisi `.env.local` saja **tidak** membuat penyimpanan online aktif. Jangan gunakan untuk penilaian sungguhan sebelum fungsi server dan pengujian keamanan selesai.

## 3. Login guru berbasis username

Supabase Auth secara bawaan menggunakan identitas seperti email/nomor telepon, bukan username bebas. Pola produksi yang disarankan:

1. Admin membuat akun Auth guru melalui dashboard/admin server (pendaftaran guru publik dimatikan).
2. Tabel `teacher_profiles` memetakan `username` unik ke `auth.users.id`.
3. Buat Edge Function `teacher-login` yang menerima username dan password melalui HTTPS, memeriksa rate limit, memetakan username ke identitas Auth, lalu melakukan autentikasi dengan alur yang aman. Jangan mengirim password ke tabel profil atau menyimpan password sendiri.
4. Dashboard memeriksa sesi Auth dan role/kelas melalui database dengan RLS.
5. Sediakan alur reset password/admin yang tidak membocorkan apakah username terdaftar.

Jangan menaruh daftar username, password, atau `service_role` key di frontend. Jangan mengandalkan hanya menyembunyikan URL dashboard.

## 4. Deploy ke Vercel

1. Buat akun di https://vercel.com.
2. Unggah proyek ke repository GitHub pribadi (jangan unggah `.env.local`).
3. Di Vercel pilih **Add New → Project**, impor repository.
4. Framework akan terdeteksi sebagai Vite; Build Command `npm run build`, Output Directory `dist`.
5. Tambahkan `VITE_SUPABASE_URL` dan `VITE_SUPABASE_ANON_KEY` pada Project Settings → Environment Variables setelah backend siap.
6. Klik Deploy. Uji URL publik yang diberikan.
7. Setiap perubahan kode dapat di-push ke repository untuk deployment ulang.

Panduan resmi Vite/Vercel: https://vercel.com/docs/frameworks/frontend/vite

## 5. Kode QR dan berbagi

Buat QR hanya setelah URL deployment production benar-benar tersedia dan bisa dibuka. Jangan gunakan `localhost`, preview URL sementara, atau alamat yang ditebak. Salin domain production dari Vercel lalu buat QR yang menunjuk ke URL tersebut. Uji QR dengan HP yang tidak sedang login ke akun pengembang.

## 6. Checklist sebelum digunakan siswa sungguhan

- [ ] Buat akun guru pertama melalui jalur admin; pendaftaran publik guru dimatikan.
- [ ] Buat kelas dan kode kelas unik, aktifkan masa berlaku bila perlu.
- [ ] Buat Edge Functions untuk join, jawab, selesai, dan login username.
- [ ] Pastikan kunci jawaban tidak dikirim ke browser sebelum jawaban dipilih.
- [ ] Pastikan RLS mencegah siswa membaca siswa lain dan guru hanya melihat kelasnya.
- [ ] Uji kode kelas salah/nonaktif/kedaluwarsa.
- [ ] Uji klik ganda, replay request, dan pengiriman skor manipulatif.
- [ ] Uji dua siswa di dua perangkat; hasil harus terlihat di dashboard guru setelah sinkronisasi.
- [ ] Uji ekspor CSV, refleksi, percobaan ulang, dan aturan retensi data.
- [ ] Tambahkan kebijakan privasi singkat dan minimalkan data pribadi siswa.
- [ ] Verifikasi backup, pemulihan, dan batas akses akun guru.

## Struktur

```text
misi-penjaga-norma/
├── src/
│   ├── App.jsx
│   ├── main.jsx
│   ├── questions.js
│   ├── style.css
│   └── supabase.js
├── supabase/
│   └── schema.sql
├── .env.example
├── index.html
├── package.json
└── vercel.json
```

## Pengujian yang sudah dapat dilakukan pada starter ini

- Halaman awal, form nama/kode kelas, empat level, 20 soal, pembahasan per soal.
- Penilaian demo dari jumlah jawaban benar, refleksi, ringkasan, detail jawaban.
- Login demo, tabel dashboard, pencarian, filter status, pengurutan skor, ekspor CSV.
- Tampilan responsif dan dukungan `prefers-reduced-motion`.

Belum diuji/diaktifkan: deployment publik, database online, login guru produksi, Edge Functions siswa, sinkronisasi antarperangkat, QR production, serta audit keamanan. Jangan menganggap fitur tersebut telah tersedia sebelum dilaksanakan dan diuji.
