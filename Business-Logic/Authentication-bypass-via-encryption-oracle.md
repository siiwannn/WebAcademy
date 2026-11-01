# Authentication Bypass via Encryption Oracle

## 📖 Konsep

Lab ini mengeksploitasi **Business Logic Vulnerability** yang menyebabkan aplikasi secara tidak sengaja menyediakan:

- **Encryption Oracle** (server mengenkripsi input kita)
- **Decryption Oracle** (server mendekripsi ciphertext yang kita kirim)

Akibatnya, attacker dapat membuat **cookie autentikasi administrator** tanpa mengetahui secret key yang digunakan server.

> **Catatan Penting**
>
> Bug ini **bukan** karena algoritma AES rusak.
>
> Yang rusak adalah **logic aplikasi** yang memperbolehkan pengguna memakai server sebagai mesin enkripsi dan dekripsi.

---

# 🧠 Apa itu Encryption Oracle?

Encryption Oracle adalah kondisi ketika server bersedia mengenkripsi **plaintext yang kita kontrol**.

Contoh:

```
administrator:timestamp
        │
        ▼
Server Encrypt
        │
        ▼
Ciphertext
```

Walaupun kita tidak mengetahui secret key, kita tetap memperoleh ciphertext hasil enkripsi server.

---

# 🧠 Apa itu Decryption Oracle?

Decryption Oracle adalah kondisi ketika server bersedia mendekripsi ciphertext yang kita kirim.

```
Ciphertext
     │
     ▼
Server Decrypt
     │
     ▼
administrator:timestamp
```

Pada lab ini hasil decrypt bahkan ditampilkan melalui error message.

---

# ⚙️ Cara Kerja Lab

## 1. Stay Logged In Cookie

Saat login menggunakan **Stay logged in**, aplikasi membuat cookie seperti:

```
stay-logged-in=Encrypted(...)
```

Awalnya isi cookie tidak diketahui.

---

## 2. Menggunakan Decryption Oracle

Notification Cookie dapat dipakai untuk mendekripsi ciphertext.

Masukkan ciphertext dari cookie:

```
stay-logged-in
```

ke

```
notification
```

Server akan mendekripsinya.

Hasil:

```
wiener:1598530205184
```

Sekarang diketahui format cookie adalah:

```
username:timestamp
```

---

## 3. Menggunakan Encryption Oracle

Saat submit komentar dengan email tidak valid:

```
administrator:timestamp
```

Server mengembalikan:

```
Set-Cookie:
notification=<ciphertext>
```

Artinya server baru saja mengenkripsi input kita.

Masalahnya...

Server otomatis menambahkan prefix:

```
Invalid email address:
```

Sehingga plaintext sebenarnya adalah:

```
Invalid email address: administrator:timestamp
```

---

# ❓ Kenapa Harus Menambah 9 Karakter?

Inilah bagian yang paling membingungkan di lab.

Prefix:

```
Invalid email address:
```

berukuran

```
23 byte
```

Sedangkan AES bekerja menggunakan blok:

```
16 byte
```

Maka pembagian blok menjadi:

```
Block 1
Invalid email ad

Block 2
dress: administr

Block 3
ator:timestamp
```

Masalahnya...

Sebagian kata

```
administrator
```

masih berada di Block 2.

Kalau Block 2 dihapus, kata administrator ikut terpotong.

---

## Solusi

Tambahkan

```
xxxxxxxxx
```

sebanyak **9 karakter**.

Sehingga:

```
23 + 9 = 32 byte
```

atau tepat:

```
2 Block AES
```

Sekarang pembagian berubah menjadi:

```
Block 1
Invalid email ad

Block 2
dress: xxxxxxxxx

Block 3
administrator:

Block 4
timestamp...
```

Sekarang administrator dimulai tepat di awal Block 3.

---

# ❓ Kenapa Tidak Langsung Hapus 23 Byte?

Karena yang dihapus pada Burp Decoder adalah:

**Ciphertext**

bukan

**Plaintext**

Ciphertext AES hanya bisa dipotong pada batas blok.

Misalnya:

Benar:

```
16 byte
16 byte
16 byte
```

Salah:

```
16 byte
9 byte
16 byte
```

Kalau menghapus 23 byte ciphertext:

```
16 byte
+
7 byte
```

maka Block kedua menjadi rusak sehingga server gagal melakukan decrypt.

Itulah alasan kita harus membuat prefix menjadi:

```
32 byte
```

agar dapat menghapus:

```
Block 1
+
Block 2
```

secara utuh.

---

# 📚 Konsep AES yang Perlu Dipahami

AES adalah **Block Cipher**.

Ukuran setiap blok:

```
16 Byte
```

Semua ciphertext selalu terdiri dari blok-blok berukuran 16 byte.

Contoh:

```
Block 1 = 16 byte
Block 2 = 16 byte
Block 3 = 16 byte
```

Jika plaintext terakhir kurang dari 16 byte, AES akan menambahkan **Padding** sebelum dienkripsi.

Contoh:

```
DATA
```

akan dipadding menjadi:

```
DATA............
```

hingga tepat 16 byte.

Karena itulah ciphertext selalu memiliki panjang kelipatan 16 byte.

---

# 🎯 Alur Serangan

1. Login menggunakan **Stay Logged In**.
2. Ambil cookie `stay-logged-in`.
3. Gunakan **Decryption Oracle** untuk mengetahui isi cookie.
4. Dapatkan format:

```
username:timestamp
```

5. Salin timestamp.
6. Gunakan **Encryption Oracle** untuk mengenkripsi:

```
xxxxxxxxxadministrator:timestamp
```

7. Decode ciphertext menggunakan Decoder.
8. Hapus **32 byte pertama** (2 Block AES).
9. Encode kembali.
10. Gunakan ciphertext tersebut sebagai:

```
stay-logged-in
```

11. Server mendekripsi menjadi:

```
administrator:timestamp
```

12. Login sebagai Administrator.
13. Akses:

```
/admin
```

14. Hapus user:

```
carlos
```

---

# 🧠 Root Cause

Bukan karena AES lemah.

Melainkan karena aplikasi:

- Menggunakan key yang sama.
- Menyediakan fitur yang mengenkripsi input user.
- Menyediakan fitur yang mendekripsi ciphertext user.
- Menggunakan hasil enkripsi tersebut sebagai cookie autentikasi.

Gabungan logic tersebut memungkinkan attacker membuat cookie administrator.

---

# 🌍 Real Bug Bounty

Kasus seperti ini **cukup jarang** ditemukan.

Alasannya karena aplikasi modern biasanya:

- Tidak memperlihatkan hasil decrypt.
- Tidak mengenkripsi input pengguna menggunakan key yang sama.
- Tidak memakai encrypted cookie sebagai mekanisme autentikasi.

Lebih sering ditemukan:

- IDOR
- SSRF
- Host Header Injection
- Business Logic
- JWT Misconfiguration
- Race Condition
- Access Control

Namun konsep yang dipelajari tetap penting.

Saat menemukan cookie atau token terenkripsi, pikirkan:

- Apakah ada endpoint yang mengenkripsi input saya?
- Apakah ada endpoint yang mendekripsi ciphertext saya?
- Apakah menggunakan secret key yang sama?
- Apakah hasil enkripsi bisa dipakai sebagai cookie atau token?

Jika jawabannya "ya", maka kemungkinan terdapat **Encryption/Decryption Oracle**.

---

# 📚 Pelajaran Lab

- Jangan hanya melihat algoritma kriptografi.
- Cari apakah aplikasi menjadi "mesin enkripsi/dekripsi" tanpa sengaja.
- Pahami bahwa AES bekerja menggunakan blok 16 byte.
- Saat memanipulasi ciphertext, penghapusan harus dilakukan per blok, bukan per karakter.
- Tujuan penambahan **9 karakter** adalah menggeser `administrator` ke awal blok berikutnya sehingga dua blok pertama dapat dihapus secara utuh.

---

# ⚡ Hafalan Cepat

```
Decrypt Cookie
        │
        ▼
Dapat Format
username:timestamp
        │
        ▼
Encrypt
xxxxxxxxxadministrator:timestamp
        │
        ▼
Tambah 9 Byte
        │
        ▼
Prefix = 32 Byte
        │
        ▼
Hapus 2 Block AES
        │
        ▼
administrator:timestamp
        │
        ▼
Jadikan Stay Logged In Cookie
        │
        ▼
Login Administrator
        │
        ▼
Delete Carlos
```

---

# 🔑 Inti Lab

**Bug Class**

```
Business Logic Vulnerability
(Authentication Bypass via Encryption Oracle)
```

**Root Cause**

```
Server menyediakan Encryption Oracle dan Decryption Oracle menggunakan secret key yang sama.
```

**AES Block Size**

```
16 Byte
```

**Trik Penting**

```
23 Byte Prefix
+
9 Byte Padding
=
32 Byte
=
2 Block AES
```

**Oracle**

```
POST /post/comment
↓
Encryption Oracle

GET /post
(notification cookie)
↓
Decryption Oracle
```

**Tujuan**

```
Membuat cookie:
administrator:timestamp
```

**Impact**

```
Authentication Bypass
→ Login sebagai Administrator
→ Menghapus user carlos
```
