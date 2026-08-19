# GraphQL — Accidental Exposure of Private GraphQL Fields

## 📖 Konsep

Lab ini menunjukkan **GraphQL field exposure** yang dikombinasikan dengan **broken access control**.

GraphQL memiliki query `getUser` yang menerima `id` dan dapat mengembalikan:

- Username
- Password

Karena ID user dapat ditebak, user biasa dapat mengambil credential administrator.

---

## ⚙️ Cara Kerja

    GraphQL Endpoint
          │
          ▼
    Introspection
          │
          ▼
    Temukan getUser
          │
          ▼
    getUser menerima ID
          │
          ▼
    ID predictable
          │
          ▼
    getUser(1)
          │
          ▼
    Administrator
          │
          ▼
    Username + Password Bocor
          │
          ▼
    Login sebagai Admin
          │
          ▼
    Delete carlos

---

## 🧠 Root Cause

Masalahnya **bukan karena GraphQL introspection aktif**.

Root cause:

- Query `getUser` dapat dipanggil oleh user biasa.
- Query menerima user-controlled ID.
- ID user predictable.
- Field sensitif `password` dapat diminta.
- Authorization tidak mencegah user mengakses credential user lain.

Intinya:

    Predictable ID
          +
    Sensitive Field
          +
    Missing Authorization
          =
    Credential Exposure

---

## 🎯 Alur Lab

1. Buka **My account** dan coba login.
2. Cari request GraphQL di **Proxy → HTTP history**.
3. Kirim request ke **Repeater**.
4. Gunakan **GraphQL → Set introspection query**.
5. Simpan query ke Site Map.
6. Cari query `getUser`.
7. Perhatikan bahwa `getUser` mengembalikan `username` dan `password`.
8. Kirim `getUser` ke Repeater.
9. Test `id=0` → tidak ada user.
10. Ubah ID menjadi `1`.
11. ID `1` adalah administrator.
12. Credential administrator terbuka.
13. Login sebagai administrator.
14. Masuk ke **Admin panel**.
15. Delete `carlos`.

---

## 🎯 Query yang Menarik

Query:

    getUser(id: $id) {
        username
        password
    }

Variables:

    {
      "id": 1
    }

Pada lab:

    ID 0 → tidak ada user
    ID 1 → administrator

---

## 🔎 Yang Dicari Saat Hunting

Jangan terpaku pada:

    getUser
    password
    administrator

Di real target, cari pola:

    GraphQL
       ↓
    Object
       ↓
    Identifier
       ↓
    Authorization
       ↓
    Sensitive Field

### Contoh Object

- `user`
- `account`
- `profile`
- `order`
- `invoice`
- `message`
- `document`
- `ticket`
- `payment`

### Contoh Sensitive Field

- `password`
- `resetToken`
- `accessToken`
- `apiKey`
- `secret`
- `internalNotes`
- `adminNotes`
- `paymentInfo`

---

## 🎯 Real Hunting

Saat menemukan GraphQL:

1. Capture request normal.
2. Pahami **Query / Mutation / Variables**.
3. Cari object yang memiliki ownership.
4. Perhatikan identifier seperti `id`, `userId`, `orderId`.
5. Jika identifier predictable, buat hypothesis **BOLA/IDOR**.
6. Gunakan **dua akun uji yang berada dalam scope** jika memungkinkan.
7. Coba Account A mengakses object milik Account B.
8. Periksa apakah sensitive field ikut terbuka.
9. Dokumentasikan impact.

Contoh:

    Account A
       │
       ▼
    order(id=1002)
       │
       ▼
    Order milik Account B
       │
       ▼
    internalNotes / private data

Jika berhasil, masalahnya adalah **authorization bypass**, bukan sekadar "ID bisa diganti".

---

## 🧪 Object-Level vs Field-Level Authorization

### Object-Level Authorization

Pertanyaan:

> Apakah Account A boleh mengakses object milik Account B?

Contoh:

    user(id=B)

Jika berhasil → indikasi **BOLA/IDOR**.

### Field-Level Authorization

Pertanyaan:

> Setelah object bisa diakses, apakah semua field di dalamnya juga boleh dibaca?

Contoh:

    username       → boleh
    email          → ?
    internalNotes  → ?
    resetToken     → ?

Field sensitif yang ikut terbuka dapat meningkatkan impact.

---

## ❗ Jangan Langsung Cari Admin

Di lab:

    ID 1 = administrator

Tetapi di real hunting jangan menjadikan **"cari ID admin"** sebagai workflow.

Lebih aman:

    Account A milik sendiri
            ↓
    Account B milik sendiri
            ↓
    Test akses A → B
            ↓
    Buktikan authorization bypass

Tidak perlu mengambil credential user sebenarnya jika bukti dengan akun uji sudah cukup.

---

## ❓ Introspection Disabled?

Jangan langsung berhenti.

Schema masih bisa dipelajari melalui:

- GraphQL request normal
- Burp HTTP history
- Frontend JavaScript
- Source map
- API documentation
- Error messages

Introspection adalah **reconnaissance**, bukan vulnerability.

    Introspection Enabled
            ≠
    Vulnerability

Sebaliknya:

    Introspection
          ↓
    Attack Surface Discovery
          ↓
    Query / Mutation / Object
          ↓
    Authorization Testing

---

## 🧪 Hypothesis Testing Saat Hunting

Gunakan pola:

    Observation
        ↓
    Hypothesis
        ↓
    Controlled Test
        ↓
    Response
        ↓
    Interpretation
        ↓
    Impact

### Observation

    GraphQL memiliki:
    order(id: ID!)

### Hypothesis

    ID mungkin dapat dimanipulasi
    untuk mengakses order user lain.

### Controlled Test

Gunakan dua akun yang berada dalam scope:

    Account A → order milik A
    Account B → order milik B

Kemudian uji:

    Account A
       ↓
    order(id=Order-B)

### Interpretation

    403 / null
    → Authorization kemungkinan bekerja

    200 + object milik A
    → Normal

    200 + object milik B
    → Potential BOLA/IDOR

Jika field sensitif juga muncul:

    Object B
       ↓
    private field
       ↓
    Sensitive Data Exposure

Impact menjadi lebih kuat.

---

## 🔬 Query vs Mutation

Jangan hanya fokus pada GraphQL **Query**.

Periksa juga **Mutation** karena authorization issue dapat terjadi pada operasi yang mengubah data.

Contoh:

    Query
      ↓
    Read unauthorized object

atau:

    Mutation
      ↓
    Modify unauthorized object

Contoh object yang menarik:

- `updateUser`
- `deleteUser`
- `updateOrder`
- `cancelOrder`
- `changeEmail`
- `changeRole`

Pertanyaan utamanya:

> Apakah user yang sedang login memang berhak melakukan operasi tersebut terhadap object tersebut?

---

## 🚨 Jangan Menganggap Semua ID Predictable sebagai Bug

ID yang predictable sendiri **belum tentu vulnerability**.

Contoh:

    /order/1001
    /order/1002
    /order/1003

Ini hanya menjadi masalah apabila:

    Predictable ID
          +
    Missing Authorization
          ↓
    Unauthorized Object Access

Jadi:

    Predictable ID ≠ Vulnerability

Sedangkan:

    Predictable ID
          +
    Broken Object Authorization
          =
    BOLA / IDOR

---

## 📚 Pelajaran Lab

- GraphQL client dapat menentukan field yang ingin diambil.
- Object ID yang predictable dapat menjadi indikator BOLA/IDOR.
- Sensitive field membutuhkan authorization yang tepat.
- Object-level authorization dan field-level authorization harus diuji.
- Introspection membantu memahami attack surface.
- Introspection sendiri bukan vulnerability.
- Jangan hanya mencari `password`; cari data sensitif secara umum.
- Query dan mutation sama-sama perlu diperiksa.
- Predictable ID sendiri belum tentu bug.
- Impact lebih penting daripada sekadar menemukan query aneh.
- Gunakan akun uji untuk membuktikan authorization bypass jika memungkinkan.

---

## ⚡ Hafalan Cepat

    GraphQL
       ↓
    Object
       ↓
    ID
       ↓
    Predictable?
       ↓
    Authorization?
       ↓
    Ownership?
       ↓
    Sensitive Field?
       ↓
    Unauthorized Access?
       ↓
    Impact

---

## 🔑 Inti Lab

### Bug Class

    Broken Access Control
            +
    GraphQL Sensitive Field Exposure

### Root Cause

    getUser()
        +
    Predictable ID
        +
    password field
        +
    Missing Authorization

### Impact

    User biasa
        ↓
    Administrator credential
        ↓
    Admin login
        ↓
    Privileged action

### Real Hunting

Jangan:

    GraphQL
       ↓
    cari password admin

Gunakan:

    Object
       ↓
    ID
       ↓
    Ownership
       ↓
    Authorization
       ↓
    Sensitive Field
       ↓
    Unauthorized Access
       ↓
    Impact

---

## 🧠 Mental Model

Saat menemukan GraphQL endpoint, jangan bertanya:

> "Apa ada password?"

Pertanyaan yang lebih berguna:

> **"Object apa yang bisa diakses oleh user ini, siapa pemiliknya, bagaimana object tersebut diidentifikasi, dan apakah authorization diterapkan sampai ke level field?"**

Dengan mental model tersebut, teknik dari lab dapat diterapkan ke berbagai object dan tidak bergantung pada query `getUser`.

---

## 📝 Checklist Hunting

- [ ] Temukan GraphQL endpoint
- [ ] Capture request normal
- [ ] Identifikasi Query dan Mutation
- [ ] Identifikasi Variables
- [ ] Cek introspection jika tersedia
- [ ] Mapping object yang tersedia
- [ ] Cari identifier
- [ ] Cek apakah identifier predictable
- [ ] Identifikasi ownership
- [ ] Uji authorization
- [ ] Gunakan dua akun uji bila memungkinkan
- [ ] Uji object-level authorization
- [ ] Uji field-level authorization
- [ ] Identifikasi sensitive data
- [ ] Hindari akses data user sebenarnya jika tidak diperlukan
- [ ] Dokumentasikan request dan response
- [ ] Tentukan impact

---

## 🎓 Kesimpulan

Lab ini bukan sekadar tentang menemukan field `password` di GraphQL.

Pelajaran utamanya adalah memahami hubungan:

    GraphQL Schema
          ↓
    Object
          ↓
    Identifier
          ↓
    Ownership
          ↓
    Authorization
          ↓
    Field Exposure
          ↓
    Impact

Dalam real hunting, fokus utama adalah **broken authorization terhadap object atau field**, sedangkan GraphQL hanya merupakan attack surface tempat masalah tersebut ditemukan.
