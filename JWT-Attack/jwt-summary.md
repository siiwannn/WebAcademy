# JWT Attack Summary (Final Fixed)

## Pola Besar Semua Bug JWT

```text
Ubah payload
↓
Signature rusak
↓
Gimana caranya server tetap percaya?
```

Semua bug JWT:
> bikin JWT yang diubah tetap dianggap valid.

## Lab → Bug → Inti

| Lab | Bug | Inti |
|---|---|---|
| 01 | Unverified signature | Server tidak cek signature |
| 02 | alg:none | Server skip verify jika alg=none |
| 03 | Weak signing key | Secret HS256 lemah |
| 04 | JWK injection | Server percaya public key di header |
| 05 | JKU injection | Server percaya URL key |
| 06 | KID path traversal | Server pakai file attacker sebagai secret |
| 07 | Algorithm confusion | Public RSA dipakai jadi secret HS256 |

---

## Bug → Terkait Apa?

| Bug | Terkait |
|---|---|
| Weak verify | Cara server verify JWT |
| Weak signing key | Secret HS256 |
| JWK | Public key |
| JKU | URL public key |
| Key confusion | RS256 → HS256 |
| KID abuse | Lookup key |
| alg:none | Mekanisme verify |
| Unverified signature | Mekanisme verify |

---

## Bug vs Algoritma

| Bug | Biasanya di | Bisa di | Kenapa |
|---|---|---|---|
| Unverified signature | HS256 / RS256 | Semua | Server tidak verify |
| alg:none | HS256 / RS256 | Semua | Server percaya `alg=none` |
| Weak signing key | HS256 | HS256 | Butuh 1 secret |
| JWK | RS256 | RSA/ECDSA | Public key |
| JKU | RS256 | RSA/ECDSA | URL key |
| Algorithm confusion | RS256→HS256 | Asymmetric→symmetric | Public key jadi secret |
| KID abuse | HS256 / RS256 | Semua | Salah lookup key |

---

## KID abuse vs KID path traversal

**KID abuse** = kategori besar

Contoh:

- path traversal
- SQLi (teoritis)
- lookup salah
- null/default key

**KID path traversal** = salah satu jenis KID abuse

```text
kid
↓
../../..
↓
baca file
↓
file jadi secret
```

Jadi:

```text
KID abuse
└── KID path traversal
```

---

## Cara Kerja Singkat

### Unverified signature
ubah payload → signature rusak → server cuek → admin

### alg:none
ubah payload → alg=none → server skip verify → admin

### Weak signing key
bruteforce secret → sign ulang → admin

### JWK
embed public key attacker → sign → admin

### JKU
kasih URL key → server download → verify → admin

### KID path traversal
ubah kid → baca file → file jadi secret → sign → admin

### Algorithm confusion
RS256 → HS256 → public key jadi secret → sign → admin

---

## HS256 vs RS256

### HS256
1 secret → sign + verify

Sering:
- Weak signing key
- Algorithm confusion (setelah ubah)
- Beberapa KID abuse

Tidak otomatis:
- alg:none
- unverified signature

### RS256
private→sign
public→verify

Sering:
- JWK
- JKU
- Algorithm confusion
- KID abuse

---

## Cara mikir lihat JWT

alg=RS256?

Hipotesis:
JWK / JKU / KID abuse / Algorithm confusion

alg=HS256?

Hipotesis:
Weak signing key / KID abuse

---

## Kategori besar bug JWT

1. Bug verify → server salah cek JWT
2. Bug key → server percaya / salah ambil key
3. Bug secret → secret lemah

## Inti akhir

```text
Semua bug JWT:
Cari cara supaya JWT yang diubah tetap valid 😭
```