# HTTP Host Header Attack --- Password Reset Poisoning

## 📖 Konsep

HTTP `Host` header digunakan server untuk mengetahui hostname tujuan
dari sebuah request.

Pada beberapa aplikasi, nilai `Host` juga digunakan untuk membentuk URL
absolut, misalnya pada fitur **reset password**, verifikasi email, atau
pembuatan link undangan.

Masalah muncul ketika server **mempercayai nilai `Host` langsung dari
request** tanpa validasi.

------------------------------------------------------------------------

## ⚙️ Cara Kerja

``` text
User meminta reset password
        │
        ▼
Server membuat link reset
        │
        ▼
https://<Host>/forgot-password?token=XXXX
        │
        ▼
Email dikirim ke korban
```

Jika `Host` berasal langsung dari request:

``` http
Host: attacker.com
```

maka server akan menghasilkan:

``` text
https://attacker.com/forgot-password?token=XXXX
```

Akibatnya token reset dapat terkirim ke domain yang dikendalikan
penyerang.

------------------------------------------------------------------------

## 🧠 Root Cause

Server menggunakan nilai `Host` dari HTTP request untuk membangun URL
tanpa memastikan bahwa hostname tersebut merupakan domain resmi
aplikasi.

``` text
Client mengontrol Host
        │
        ▼
Server percaya
        │
        ▼
Host digunakan membuat URL
```

------------------------------------------------------------------------

## 🎯 Alur Serangan

``` text
Korban meminta reset password
        │
        ▼
Attacker mengubah Host header
        │
        ▼
Server membuat link reset menggunakan Host palsu
        │
        ▼
Token reset dikirim ke domain attacker
        │
        ▼
Attacker memperoleh token
        │
        ▼
Password korban diubah
        │
        ▼
Account Takeover
```

------------------------------------------------------------------------

## 💥 Impact

-   Password Reset Poisoning
-   Pencurian reset token
-   Account Takeover (ATO)
-   Pengambilalihan akun pengguna

------------------------------------------------------------------------

## 🛡️ Mitigasi

-   Jangan menggunakan `Host` dari request untuk membuat URL absolut.
-   Gunakan base URL yang telah dikonfigurasi di server.
-   Validasi `Host` menggunakan allowlist.
-   Gunakan proxy/web server yang hanya menerima hostname yang sah.

------------------------------------------------------------------------

## ⚡ Hafalan Cepat

``` text
Host Header dipercaya
        │
        ▼
Server membuat link reset
        │
        ▼
Host dapat dimanipulasi
        │
        ▼
Token reset bocor
        │
        ▼
Account Takeover
```
