# GraphQL — Accessing Private Posts

## Konsep

Lab ini menunjukkan vulnerability pada GraphQL ketika:

1. Ada object/post yang seharusnya private.
2. ID object dapat ditebak atau ditemukan.
3. GraphQL mengizinkan akses langsung ke object tersebut.
4. Schema GraphQL mengekspos field sensitif.
5. Field sensitif tersebut dapat diminta oleh user yang tidak seharusnya memiliki akses.

Di lab ini field sensitifnya adalah `postPassword`.

**Catatan:** `postPassword` hanya contoh dari lab. Di real target, field sensitif bisa memiliki nama lain seperti `password`, `passwordHash`, `resetToken`, `accessToken`, `apiKey`, `secret`, `internalNotes`, atau `privateData`.

Jadi jangan menghafal `postPassword`. Yang harus dipahami adalah **mencari sensitive field yang bisa diakses melewati authorization boundary**.

---

## Mental Model

GraphQL endpoint  
↓  
Lihat query normal  
↓  
Perhatikan object / ID  
↓  
ID sequential / predictable  
↓  
Ada ID yang hilang  
↓  
Hypothesis: ada hidden/private object  
↓  
GraphQL introspection  
↓  
Cari field yang tersedia  
↓  
Temukan sensitive field  
↓  
Request object menggunakan ID tersebut  
↓  
Tambahkan sensitive field  
↓  
Server membocorkan data

Intinya:

> Jangan hanya melihat data yang diberikan aplikasi. Perhatikan object yang seharusnya ada, schema yang tersedia, dan field sensitif yang dapat diminta oleh client.

---

## Observation dari Lab

Saat membuka blog, request GraphQL mengambil daftar blog post.

Response menunjukkan ID:

`1, 2, 4`

ID `3` tidak muncul.

Ini belum membuktikan vulnerability.

Tetapi merupakan **indikasi** bahwa object dengan ID `3` mungkin masih ada tetapi sengaja tidak ditampilkan pada listing.

---

## Hypothesis

Dari observation:

`1 → 2 → 4`

muncul hypothesis:

> ID 3 mungkin merupakan hidden/private blog post.

Maka kita mencoba mengakses object tersebut secara langsung.

Ini memiliki kemiripan dengan pola IDOR/BOLA. Jika aplikasi hanya menyembunyikan object dari UI tetapi endpoint/API tetap dapat mengaksesnya, terdapat kemungkinan authorization flaw.

---

## Menemukan GraphQL Endpoint

Di Burp Suite:

`Proxy → HTTP history`

Cari request:

`POST /graphql/v1`

Kemudian:

`Right click → Send to Repeater`

---

## GraphQL Introspection

GraphQL mempunyai fitur **introspection**.

Introspection memungkinkan client mengetahui struktur/schema GraphQL.

Secara sederhana:

Client  
↓  
"Apa saja type dan field yang tersedia?"  
↓  
GraphQL  
↓  
Schema

Di Burp Repeater:

`Right click pada Request panel → GraphQL → Set introspection query`

Kemudian kirim request.

Cari type:

`BlogPost`

Perhatikan field yang tersedia.

Dalam lab ditemukan:

- `id`
- `title`
- `content`
- `postPassword`

Field yang menarik adalah:

`postPassword`

---

## Kenapa Introspection Penting?

**Introspection bukan otomatis vulnerability.**

Contohnya, jika:

`Introspection enabled → Schema dapat dilihat`

itu belum tentu berarti ada bug.

Yang menarik adalah ketika schema memberikan petunjuk bahwa terdapat field sensitif seperti:

`BlogPost → postPassword`

Kemudian field tersebut ternyata dapat diminta oleh user yang tidak seharusnya mendapatkannya.

Jadi vulnerability muncul dari kombinasi:

`Predictable object + Improper authorization + Sensitive field exposure`

---

## Mengakses Hidden Post

Kembali ke request GraphQL normal.

Pada GraphQL tab terdapat bagian:

- Query
- Variables

Variables awal misalnya:

`{"id": 1}`

Karena sebelumnya ditemukan ID `3` yang hilang, ubah menjadi:

`{"id": 3}`

Tujuannya adalah mencoba mengakses hidden/private object.

---

## Menambahkan Sensitive Field

Query normal mungkin hanya meminta:

`blogPost(id: $id) { id title content }`

Karena introspection menunjukkan bahwa `BlogPost` mempunyai field `postPassword`, field tersebut dapat ditambahkan:

`blogPost(id: $id) { id title content postPassword }`

Variables:

`{"id": 3}`

Kirim request.

Jika server memberikan response yang berisi:

`"postPassword": "..."`

berarti password private post berhasil diperoleh.

Password tersebut kemudian digunakan untuk menyelesaikan lab.

---

## Apa yang Sebenarnya Terjadi?

Vulnerability chain:

Hidden post  
↓  
ID = 3  
↓  
ID dapat ditebak  
↓  
GraphQL menerima ID tersebut  
↓  
Schema menunjukkan adanya `postPassword`  
↓  
Client meminta `postPassword`  
↓  
Server mengembalikan password

Masalah utamanya bukan sekadar:

> "`postPassword` ada."

Masalahnya adalah:

> User yang tidak seharusnya memiliki akses dapat meminta field sensitif dan server memberikannya.

---

## `postPassword` di Real-World Hunting

Jangan mencari field bernama `postPassword` karena itu hanya nama field pada lab.

Di real target, cari **sensitive fields**.

Contohnya:

- `password`
- `passwordHash`
- `resetToken`
- `accessToken`
- `refreshToken`
- `apiKey`
- `secret`
- `privateKey`
- `internalNotes`
- `adminNotes`
- `paymentInfo`

Tetapi nama field bisa berbeda.

Contohnya:

- `credential`
- `auth`
- `key`
- `secretValue`
- `privateData`
- `securityToken`

Jadi yang dicari bukan nama tertentu.

Yang dicari adalah:

> **Apakah GraphQL memberikan data yang seharusnya berada di luar authorization boundary user?**

---

## Real-World Hunting Workflow

### 1. Identifikasi endpoint

Cari endpoint seperti:

- `POST /graphql`
- `POST /graphql/v1`
- `POST /api/graphql`

atau endpoint GraphQL lainnya.

### 2. Lihat query normal

Perhatikan:

- Object
- ID
- Variables
- Fields
- Arguments

Contoh konsep:

`user(id: $id) { id username email }`

### 3. Perhatikan ID

Cari pola seperti:

`1, 2, 3, 4`

atau:

`1001, 1002, 1003`

Jika predictable, muncul pertanyaan:

> Apakah object lain bisa diakses?

### 4. Cari object yang menarik

Contohnya:

- `user`
- `order`
- `invoice`
- `message`
- `post`
- `document`
- `ticket`
- `payment`
- `profile`

Object-object tersebut sering memiliki authorization boundary.

### 5. Periksa schema

Jika introspection tersedia, lihat:

- Types
- Fields
- Arguments
- Mutations
- Relationships

Cari field yang berpotensi sensitif.

### 6. Test authorization

Contoh konsep:

`Account A → mencoba mengakses object milik Account B`

Gunakan akun/data yang memang berada dalam scope dan, bila memungkinkan, object milik akun uji sendiri.

### 7. Periksa sensitive fields

Jika object dapat diakses, lihat apakah server juga memberikan data seperti:

- email
- privateData
- token
- internalNotes
- dan data sensitif lainnya

---

## Observation → Interpretation → Test

Gunakan pola ini saat hunting.

### Observation

- GraphQL endpoint ditemukan.
- Object menggunakan sequential ID.
- Ada ID yang tidak muncul pada listing.
- Schema memiliki field yang terlihat sensitif.

### Interpretation

- Mungkin terdapat hidden object.
- Mungkin object dapat diakses secara langsung.
- Mungkin authorization hanya diterapkan pada frontend/listing.
- Mungkin sensitive field dapat diakses melalui GraphQL.

### Test

- Request object dengan ID lain.
- Bandingkan response.
- Coba field yang relevan.
- Periksa apakah authorization diterapkan pada object dan field.

---

## Interpretasi Hasil

### Case 1 — ID tidak bisa diakses

Jika request ID 3 menghasilkan `403` atau `unauthorized`, kemungkinan authorization bekerja.

Biasanya berhenti, kecuali ada jalur lain yang masih relevan untuk diuji.

### Case 2 — ID dapat diakses tetapi hanya data publik

Jika ID 3 hanya memberikan `title` dan `content`, belum tentu vulnerability.

Cari tahu apakah object tersebut memang seharusnya public.

### Case 3 — Private object dapat diakses

Jika ID 3 memberikan private content, ini jauh lebih menarik.

Periksa authorization dan impact.

### Case 4 — Sensitive field dapat dibaca

Jika ID 3 memberikan `postPassword` atau data sensitif lainnya, ini merupakan indikasi kuat sensitive data exposure / authorization flaw.

### Case 5 — Introspection disabled

Jika introspection menghasilkan error, jangan langsung berhenti.

Schema masih bisa dipelajari dari:

- Frontend JavaScript
- Normal GraphQL requests
- Burp HTTP history
- API documentation
- Error messages
- Known application behavior

Introspection hanya salah satu cara memahami schema.

---

## Kapan Lanjut dan Kapan Stop?

### Lanjut jika:

- Object ID predictable.
- Ada object yang tidak muncul pada listing.
- Object lain bisa diakses.
- Authorization terlihat tidak konsisten.
- Ada sensitive field.
- Sensitive data dapat dibaca tanpa privilege yang sesuai.

### Stop jika:

- Authorization bekerja dengan benar.
- Object tidak dapat diakses.
- Field sensitif tetap dilindungi.
- Tidak ada impact yang nyata.

Jangan terus melakukan fuzzing field secara agresif kalau sudah jelas authorization bekerja.

---

## Kesalahan Pemahaman

### Salah

> GraphQL introspection enabled = vulnerability.

Tidak selalu.

### Salah

> Harus mencari `postPassword`.

Tidak.

`postPassword` hanya contoh dari lab.

### Yang benar

> Saya mencari object dan field yang seharusnya tidak dapat diakses oleh privilege saya.

---

## GraphQL dan IDOR/BOLA

Tekniknya memiliki kemiripan dengan IDOR/BOLA.

REST:

`GET /api/user/123`

Kemudian mencoba:

`123 → 124`

GraphQL:

`user(id: 123) { id username }`

Kemudian mencoba:

`user(id: 124) { id username }`

Perbedaannya adalah GraphQL memungkinkan client menentukan **field** yang ingin dikembalikan.

Jadi attack surface-nya dapat mencakup:

- Object authorization
- Field-level authorization
- Query authorization
- Mutation authorization

Ini yang membuat GraphQL menarik untuk diuji.

---

## Rangkuman Lab

1. Buka blog.
2. Intercept GraphQL request.
3. Lihat response.
4. Perhatikan ID sequential.
5. Temukan ID yang hilang → `3`.
6. Kirim request ke Repeater.
7. Jalankan GraphQL introspection.
8. Cari `BlogPost` type.
9. Temukan field `postPassword`.
10. Ubah variable ID menjadi `3`.
11. Tambahkan `postPassword` ke query.
12. Kirim request.
13. Password bocor.
14. Submit password ke lab.

---

## Inti yang Harus Nempel

> **GraphQL bukan sekadar endpoint API. Client bisa menentukan field yang ingin diambil.**

Ketika menemukan GraphQL endpoint saat hunting:

GraphQL  
↓  
Object apa yang tersedia?  
↓  
Object mana yang bisa ditebak?  
↓  
Authorization-nya benar?  
↓  
Field apa yang tersedia?  
↓  
Ada data sensitif?  
↓  
Apakah user biasa bisa mengambilnya?

Jangan menghafal:

`postPassword`

Yang harus dihafal adalah **alur reasoning**:

Observation  
↓  
Hypothesis  
↓  
Test authorization  
↓  
Inspect schema  
↓  
Identify sensitive field  
↓  
Verify unauthorized access  
↓  
Assess impact

---

## One-Liner

> **Cari object yang bisa diakses secara tidak semestinya, pahami schema GraphQL, lalu cek apakah field sensitif pada object tersebut juga dapat diambil tanpa authorization yang sesuai.**
