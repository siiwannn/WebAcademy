# JWT Authentication Bypass via JWK Injection

## Konsep

### Apa itu `jwk`?

`jwk` = **JSON Web Key**

Fungsi:

```text
Menyimpan public key langsung di header JWT
```

Contoh:

```json
{
 "jwk":{
    "kty":"RSA",
    "kid":"893d...",
    "n":"...",
    "e":"AQAB"
 }
}
```

Server:

```text
Lihat jwk
↓
Ambil public key dari JWT
↓
Verify JWT
```

---

### Kenapa bug bisa terjadi?

Normal:

```text
Server hanya percaya public key internal
↓
Verify JWT
```

Bug:

```text
Server percaya public key dari JWT attacker
↓
Verify pakai key attacker
↓
JWT palsu dianggap valid
```

Inti bug:

> Server membiarkan attacker menentukan public key untuk verifikasi JWT.

---

### Apa itu `kid`?

`kid` = **Key ID**

Fungsi:

```text
Menentukan key mana yang dipakai
```

Kadang perlu diubah supaya cocok dengan key attacker.

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

## 2. Edit Payload

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

## 3. Embed JWK

Klik:

```text
Attack
↓
Embedded JWK
↓
Pilih RSA key attacker
```

Burp otomatis:

```text
Masukkan public_attacker
↓
ke header JWT
```

Header jadi:

```json
{
 "alg":"RS256",
 "jwk":{
    public_attacker
 }
}
```

Kadang:

```json
"kid":"893d..."
```

juga ikut diubah.

---

## 4. Sign JWT

JWT ditandatangani:

```text
private_attacker
```

---

# Cara Server Memproses

```text
Terima JWT
↓
Lihat jwk
↓
Ambil public_attacker dari header
↓
Verify signature
↓
Valid
↓
sub=administrator
↓
Admin access
```

---

# Hafalan Cepat

```text
Generate key
↓
sub=administrator
↓
Embed JWK
↓
Sign
↓
Server pakai public_attacker
↓
JWT valid
↓
Admin
```

---

# Inti Lab

> JWK Injection = attacker menyisipkan public key sendiri ke header JWT lalu server mempercayainya.
