# JWT Authentication Bypass via KID Header Path Traversal

## Konsep

`kid` = Key ID.

Normal:

    kid -> pilih secret -> verify

Bug:

    kid -> path traversal -> baca file -> jadi secret

## Inti bug

> Attacker bisa menentukan file mana yang dipakai server sebagai secret
> JWT.

## Kenapa New Symmetric Key?

Karena JWT tetap butuh signature valid.

    ubah payload
    ↓
    signature rusak
    ↓
    harus sign ulang

HS256 = 1 secret → sign + verify.

## Kenapa `/dev/null`?

Server:

    secret = read('/dev/null')
    ↓
    secret=''

Tujuan exploit: \> paksa server pakai secret yang attacker tahu.

## Kenapa `AA==`?

`AA==` decode → `\x00` (NULL byte), bukan empty string. Dipakai karena
Burp tidak mau sign pakai secret kosong.

**Workaround = jalan muter / solusi sementara**.

## Real world

Tidak harus `/dev/null`. Cari file yang: - ada - predictable - pendek -
stabil

Contoh: `/etc/hostname`, config file, test secret.

## Pertanyaan penting

**Masa isi file panjang dijadiin secret?** Secara konsep: iya 😭 kalau
attacker tahu isi file.

## Hafalan

    kid
    ↓
    path traversal
    ↓
    server baca file attacker pilih
    ↓
    pakai sebagai secret
    ↓
    attacker sign pakai secret sama
    ↓
    JWT valid
