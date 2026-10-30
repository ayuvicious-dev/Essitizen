# Esitizen – Persiapan Pindah Kewarganegaraan

Aplikasi web (PWA) untuk memantau persiapan pindah kewarganegaraan. Login memakai **Firebase Authentication** (Email/Password), data tersimpan di **Cloud Firestore**, dan bisa dipasang di layar utama ponsel serta dibuka offline.

Menu bawah:

| Menu | Isi |
|---|---|
| Dashboard | Skor kesiapan keseluruhan, negara utama + hitung mundur, langkah berikutnya, ringkasan pekan ini |
| Negara | Daftar negara tujuan (katalog 11 negara + negara kustom), status, tanggal target, catatan |
| Persiapan | **Bahasa** (tes, skor, kemampuan, log belajar), **Finansial** (target dana, rincian biaya, perkiraan waktu tercapai), **Mental** (cek suasana hati, jurnal, checklist) |
| Setting | Profil, tema, mata uang, pasang aplikasi, ekspor data, reset sandi, keluar, hapus akun |

## Struktur file

```
icons/               ikon PWA (dibuat dari Blue_Growth_Profile_Icon.png)
index.html           seluruh aplikasi (HTML + CSS + JS)
manifest.json        metadata PWA
service-worker.js    cache offline
README.md
```

## Langkah setup

### 1. Isi konfigurasi Firebase
Firebase Console ▸ **Project settings** ▸ **General** ▸ **Your apps** ▸ tambahkan **Web app** (`</>`) ▸ salin objek `firebaseConfig`.
Buka `index.html`, cari `const firebaseConfig`, lalu ganti nilai `ISI_API_KEY`, `ISI_SENDER_ID`, dan `ISI_APP_ID` (nilai lain sudah diisi sesuai project `essitizen`; samakan jika berbeda).

### 2. Authentication
Di screenshot Anda, provider **Email/Password** sudah *Enabled*. Tidak ada yang perlu diubah.
Setelah aplikasi di-hosting, tambahkan domainnya di **Authentication ▸ Settings ▸ Authorized domains** (mis. `username.github.io`). Domain `*.firebaseapp.com` dan `localhost` sudah otomatis.

### 3. Aturan Firestore (WAJIB diganti)
Aturan Anda saat ini `allow read, write: if false;`, sehingga semua simpan/baca akan ditolak.
Buka **Firestore Database ▸ Rules**, ganti dengan aturan berikut, lalu **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

Setiap pengguna hanya bisa membaca dan menulis dokumennya sendiri (`users/{uid}`).

### 4. Jalankan / deploy
Service worker dan login butuh **HTTPS** atau `localhost`; membuka `index.html` langsung dengan klik ganda tidak akan berfungsi penuh.

- **Lokal:** `python3 -m http.server 8080` di folder ini, lalu buka `http://localhost:8080`.
- **GitHub Pages:** unggah semua file ke repo, aktifkan Pages. Semua path sudah relatif.
- **Firebase Hosting:** `firebase init hosting` (public directory = folder ini, single-page = No), lalu `firebase deploy`.

## Struktur data (Firestore)

Satu dokumen per pengguna: `users/{uid}`

```
name, theme, currency
countries[]  { id, name, flag, years, lang, dual, status, target, notes, priority }
prep.lang    { target, test, currentScore, targetScore, testDate, hoursWeek, skills{...}, log[], tasks[] }
prep.fin     { goal, saved, monthly, costs[], tasks[] }
prep.mental  { moods[], journal, tasks[] }
updatedAt
```

## Catatan

- Setelah mengubah file aplikasi, naikkan `VERSION` di `service-worker.js` (mis. `v2`) agar cache perangkat pengguna diperbarui.
- Ringkasan syarat tiap negara di katalog bersifat **umum dan bisa berubah**. Selalu cek situs imigrasi resmi sebelum mengambil keputusan.
- Indonesia tidak mengakui kewarganegaraan ganda bagi orang dewasa (UU 12/2006). Aplikasi menampilkan pengingat ini di menu Negara.
- Nama aplikasi bisa diganti di `index.html` (`<title>`, teks login) dan `manifest.json` (`name`, `short_name`).
