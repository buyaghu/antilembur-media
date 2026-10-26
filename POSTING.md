# Prosedur posting otomatis @antilembur.ai

Dijalankan oleh scheduled task Claude setiap hari sekitar pukul 11.00 WIB.
Ikuti langkah ini persis. Jangan memposting apa pun di luar prosedur ini.

## Aturan keras
- Hanya posting entri `jadwal.json` dengan `status` = `"siap"` dan `tanggal` = hari ini (zona Asia/Jakarta).
- Entri berstatus `draft` belum disetujui pemilik akun. Jangan diposting, jangan diubah.
- Maksimal satu posting per hari. Kalau ada lebih dari satu entri `siap` untuk hari ini, posting yang pertama saja.
- Jangan memposting ulang entri berstatus `tayang`.
- **Cegah posting ganda lewat Instagram, bukan hanya lewat status.** Sebelum membuat container, ambil 5 media terakhir (`INSTAGRAM_GET_IG_USER_MEDIA`). Kalau sudah ada postingan dengan baris pertama caption yang sama dengan entri hari ini, anggap sudah tayang: jangan posting, cukup laporkan. (Sesi terjadwal kadang tidak bisa push ke repo ini, jadi `status` di GitHub bisa tertinggal.)
- Akun Instagram: koneksi Composio dengan alias `antilembur` (username `antilembur.ai`, ig_user_id `29112358381728688`). Sebelum posting, cek dengan `INSTAGRAM_GET_USER_INFO` bahwa username-nya `antilembur.ai`. Kalau bukan, berhenti.

## Langkah
1. Clone `https://github.com/buyaghu/antilembur-media` (repo publik, clone biasa cukup) atau baca `jadwal.json` langsung dari `https://raw.githubusercontent.com/buyaghu/antilembur-media/main/jadwal.json`.
2. Baca `jadwal.json` dan cari entri sesuai aturan di atas. Kalau tidak ada, selesai tanpa mengubah apa pun.
3. Pastikan `https://raw.githubusercontent.com/buyaghu/antilembur-media/main/<video>` mengembalikan HTTP 200.
4. Pakai caption dari `jadwal.json` apa adanya. **Jangan** mengganti `#` dengan `%23`: Composio sudah meng-encode sendiri, dan `%23` akan tampil apa adanya di Instagram (terbukti di ep01).
5. Composio `INSTAGRAM_POST_IG_USER_MEDIA` (akun `antilembur`): `ig_user_id`, `video_url` (URL raw di atas), `media_type` = `REELS`, `share_to_feed` = `true`, `caption`.
6. Composio `INSTAGRAM_POST_IG_USER_MEDIA_PUBLISH`: `creation_id` dari langkah 5, `max_wait_seconds` = 300.
   - Kalau gagal karena video masih diproses, tunggu lalu publish lagi dengan `creation_id` yang sama.
   - Kalau container berstatus `ERROR`, buat container baru satu kali saja lalu publish.
   - Sebelum mencoba ulang, cek `INSTAGRAM_GET_IG_USER_MEDIA` (limit 5) supaya tidak terjadi posting ganda.
7. Ambil `permalink` dan `caption` lewat `INSTAGRAM_GET_IG_MEDIA`.
8. Perbarui entri di `jadwal.json`:
   - Berhasil: `status` = `"tayang"`, tambah `media_id`, `permalink`, `waktu_tayang` (ISO, WIB). Kalau caption yang tampil berbeda dari caption di jadwal (misalnya hashtag hilang atau muncul `%23`), tambah `catatan` yang menjelaskan bedanya.
   - Gagal: `status` = `"gagal"`, tambah `catatan` berisi pesan error.
9. Commit (`Posting <episode>: <status>`) dan coba push ke `main`. Kalau push ditolak (misalnya 403), itu tidak apa-apa: jangan dicoba lewat cara lain, cukup sebutkan di laporan supaya status dicatat dari sesi lain.
10. Laporkan hasil singkat: episode, status, permalink atau error.
