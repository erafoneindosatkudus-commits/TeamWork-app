# TeamWork — Aplikasi Android untuk Kerja Tim

Aplikasi Android (Kotlin + Jetpack Compose + Firebase) untuk kolaborasi tim.

## Fitur

- **Akun & tim**: daftar/masuk dengan email, buat tim, atau gabung lewat kode undangan 6 karakter.
- **Papan tugas**: tambah, **edit** (ketuk kartu), pindah status (Akan dikerjakan → Dikerjakan → Selesai), hapus; ada penanggung jawab dan tenggat.
- **Chat tim**: pesan real-time.
- **Kalender**: jadwal rapat/acara; tenggat tugas ikut tampil di agenda.
- **Berbagi file**: unggah, buka, hapus.
- **Notifikasi push (FCM)**: pesan, tugas, dan jadwal baru dikirim ke semua anggota tim (pengirim tidak dinotifikasi).
- **Pengingat tenggat**: notifikasi lokal 1 jam sebelum tugas/jadwal (WorkManager, tanpa server).
- **Ikon aplikasi** (adaptive icon) dan ikon notifikasi.

---

## A. Langsung jadi APK (tanpa Android Studio) — GitHub Actions

1. **Siapkan Firebase** (sekali saja, lihat bagian C) dan unduh `google-services.json`.
2. Buat repository GitHub, unggah seluruh isi folder ini (termasuk folder `.github`).
3. Ubah `google-services.json` menjadi base64:
   - Linux/Mac: `base64 -w0 app/google-services.json`
   - Windows PowerShell: `[Convert]::ToBase64String([IO.File]::ReadAllBytes("app\google-services.json"))`
4. Di GitHub: **Settings → Secrets and variables → Actions → New repository secret**,
   nama `GOOGLE_SERVICES_JSON`, isi dengan hasil base64 tadi.
5. Buka tab **Actions → Build APK → Run workflow** (juga jalan otomatis tiap push).
6. Setelah selesai (±5 menit), unduh **TeamWork-debug-apk** dari bagian *Artifacts*,
   ekstrak, lalu pasang `app-debug.apk` di HP (izinkan "instal dari sumber tidak dikenal").

## B. Build lewat Android Studio

Buka folder ini di Android Studio → letakkan `google-services.json` di `app/` →
**Build → Build APK(s)**, atau jalankan langsung ke perangkat/emulator (Android 7.0+).
Lewat terminal: `./gradlew :app:assembleDebug` → hasil di `app/build/outputs/apk/debug/`.

## C. Setup Firebase (wajib)

1. [Firebase Console](https://console.firebase.google.com) → buat proyek → **Add app → Android**,
   package name `com.example.teamwork` → unduh `google-services.json` ke folder `app/`.
2. Aktifkan: **Authentication → Email/Password**, **Firestore Database**, **Storage**.
3. Tempel `firestore.rules` ke Firestore → Rules, dan `storage.rules` ke Storage → Rules, lalu Publish.
4. Pada Firestore, query tenggat memakai indeks satu-kolom otomatis; tidak perlu indeks tambahan.

## D. Aktifkan notifikasi push dari server (opsional, perlu paket Blaze)

Pengingat tenggat (lokal) langsung bekerja tanpa langkah ini. Untuk notifikasi pesan/tugas/jadwal
baru saat aplikasi tertutup, deploy Cloud Functions:

```
npm install -g firebase-tools
firebase login
firebase use --add          # pilih proyek Anda
cd functions && npm install && cd ..
firebase deploy --only functions
```

Cloud Functions membutuhkan paket **Blaze** (bayar sesuai pemakaian; ada kuota gratis yang cukup untuk tim kecil).

## Alur pemakaian

1. Daftar → **Buat tim** atau **Gabung** dengan kode.
2. Bagikan kode lewat menu ⋮ → *Kode undangan tim*.
3. Izinkan notifikasi saat diminta (Android 13+).

## Struktur kode

```
app/src/main/java/com/example/teamwork/
├── MainActivity.kt
├── data/            # Models.kt, TeamRepository.kt (Auth, Firestore, Storage)
├── notify/          # Notifier, TeamMessagingService (FCM), ReminderWorker
└── ui/              # AppViewModel, TeamWorkApp, Util, theme, screens/
functions/           # Cloud Functions pengirim push
.github/workflows/   # build-apk.yml
```

Data Firestore:

```
users/{uid}                    { name, email, teamId }
teamCodes/{kode}               { teamId }
teams/{teamId}                 { name, code, ownerUid }
teams/{teamId}/tasks|messages|events|files/{id}
```

## Catatan

- Proyek ini ditulis tanpa Android SDK di lingkungan pembuatnya, jadi **belum pernah dikompilasi**.
  Jika build gagal, buka log Actions / Android Studio dan sesuaikan versi dependensi di `build.gradle.kts`.
- APK yang dihasilkan adalah **debug** (ditandatangani kunci debug) — cukup untuk dipasang langsung.
  Untuk rilis Play Store, buat keystore dan build `assembleRelease`.
- Aturan keamanan bersifat dasar: siapa pun yang tahu kode undangan dapat bergabung.
