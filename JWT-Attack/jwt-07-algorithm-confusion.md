# JWT Authentication Bypass via Algorithm Confusion

## Konsep Inti
RS256: private->sign, 
             public->verify.

Exploit:
RS256 -> HS256 -> pakai public key server sebagai secret HS256.

## Kenapa ubah alg?
Tetap RS256 = butuh private key server (gak punya).
Makanya ubah ke HS256.

## Kenapa New RSA Key?
Import public key server (JWK), bukan generate private.

## Apa itu PEM?
Format public key:
-----BEGIN PUBLIC KEY----- ...

## Kenapa Copy as PEM?
Ubah format public key supaya bisa dipakai sebagai secret.

## Kenapa Base64?
Karena field k butuh Base64.

## Apa itu k?
k = key = secret HS256.

## Kenapa New Symmetric Key?
Karena HS256 butuh 1 secret.
Isi secret = public key server.

## Pertanyaan penting
'Kenapa gak langsung ubah sub?' -> signature rusak, perlu sign ulang.
'Kenapa gak random secret?' -> server verify pakai public_server, harus sama.

## Real case
Lihat alg=RS256 -> cari /jwks.json -> hipotesis -> tes.
Mayoritas gagal.

## Hafalan
RS256 -> dapat public key -> ubah HS256 -> public key jadi secret -> sign -> admin.

## Inti
Algorithm confusion = server salah pakai public key RSA sebagai secret HS256.