JWT Authentication Bypass via Algorithm Confusion (No Exposed Key)

Konsep

Lab ini sebenarnya masih menggunakan bug:

Algorithm Confusion

Bedanya dengan lab sebelumnya:

Lab 07

Public key tersedia
↓
/jwks.json
↓
langsung exploit

Lab 08

Public key tidak tersedia
↓
harus dicari dulu
↓
baru exploit

---

Inti Bug

Server normal:

RS256

private key -> sign
public key -> verify

Attacker:

punya JWT
↓
tidak punya public key
↓
harus mendapatkannya dulu

Setelah public key didapat:

RS256
↓
HS256
↓
public key jadi secret
↓
sign JWT

Persis seperti lab Algorithm Confusion sebelumnya.

---

Kenapa Perlu 2 JWT?

Login:

JWT 1

Logout lalu login lagi:

JWT 2

Kenapa?

Karena tool:

sig2n

membutuhkan:

minimal 2 signature berbeda

untuk menghitung kemungkinan:

RSA modulus (n)

yang dipakai server.

---

Apa itu sig2n?

Tool:

sig2n

digunakan untuk:

JWT 1
+
JWT 2
↓
hitung kandidat public key

Output:

Public Key A
JWT A

Public Key B
JWT B

Kadang hanya:

1 kandidat

Kadang lebih.

---

Kenapa Docker?

Docker:

bukan exploit

Docker hanya:

cara menjalankan sig2n

Kalau tool tersedia langsung:

sig2n token1 token2

hasilnya sama.

---

Kenapa Tool Menghasilkan JWT Palsu?

Ini bagian yang bikin bingung 😭

JWT palsu tersebut:

BUKAN JWT admin

Tujuannya:

mengetes kandidat public key

Karena:

Public Key A ?
Public Key B ?

belum tahu mana yang benar.

---

Cara Menentukan Public Key yang Benar

Output tool:

Public Key A
JWT A

Coba:

JWT A
↓
/my-account

Kalau:

200 OK

berarti:

Public Key A benar

Kalau:

302 Redirect Login

berarti:

Public Key A salah

Lanjut ke kandidat berikutnya.

---

Kenapa New Symmetric Key?

Karena nanti:

RS256
↓
HS256

HS256 membutuhkan:

1 secret

Maka:

New Symmetric Key

---

Apa Isi Field k?

Isi:

Base64(public key PEM)

Artinya:

secret HS256
=
public key server

---

Langkah Exploit

1. Ambil 2 JWT

Login
↓
JWT 1

Logout
↓
Login
↓
JWT 2

---

2. Jalankan sig2n

docker run --rm -it portswigger/sig2n <JWT1> <JWT2>

---

3. Cari Public Key yang Benar

Output:

Public Key
JWT Testing

Tes JWT tersebut.

Kalau:

200 OK

maka public key benar.

---

4. Buat Symmetric Key

Burp:

JWT Editor
↓
New Symmetric Key

Isi:

k = Base64(public key PEM)

---

5. Edit JWT

Header:

{
 "alg":"HS256"
}

Payload:

{
 "sub":"administrator"
}

---

6. Sign JWT

Sign
↓
Pilih Symmetric Key

---

7. Akses Admin

/admin
↓
Delete carlos

---

Pertanyaan Penting 😭

"Bukannya ini sama dengan lab sebelumnya?"

IYA.

Bug:

Algorithm Confusion

masih sama.

Yang berbeda:

cara mendapatkan public key

---

"Kenapa harus pakai Docker?"

Tidak harus.

Docker cuma:

cara menjalankan sig2n

---

"Kenapa tool menghasilkan JWT palsu?"

Untuk:

mengetes kandidat public key

bukan untuk langsung jadi admin.

---

"Kenapa perlu 2 JWT?"

Karena:

JWT 1 + JWT 2
↓
sig2n
↓
hitung kandidat modulus RSA
↓
public key

---

Hafalan Cepat

2 JWT
↓
sig2n
↓
dapat kandidat public key
↓
tes JWT hasil tool
↓
ketemu public key benar
↓
RS256 -> HS256
↓
public key jadi secret
↓
sub=administrator
↓
sign
↓
admin

---

Inti Lab

«Lab ini bukan mengajarkan bug JWT baru.»

Yang baru adalah:

cara mendapatkan public key

Sedangkan exploit akhirnya tetap:

Algorithm Confusion
↓
public key jadi secret HS256
