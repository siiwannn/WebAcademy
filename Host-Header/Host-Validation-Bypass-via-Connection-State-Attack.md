# Host Validation Bypass via Connection State Attack

## 📖 Konsep

Lab ini mengeksploitasi **Routing-Based SSRF** yang terjadi karena **Front-end Server menyimpan Connection State**.

Bug bukan berada pada backend, melainkan pada **Front-end / Reverse Proxy** yang hanya melakukan validasi **Host** pada request pertama dalam satu koneksi TCP.

Request selanjutnya pada koneksi TCP yang sama dianggap masih berasal dari Host yang sudah tervalidasi.

Akibatnya attacker dapat mengakses resource internal seperti:

```
192.168.0.1/admin
```

walaupun Host tersebut seharusnya ditolak.

---

# ⚙️ Cara Kerja

```
TCP Connection Dibuat
        │
        ▼
Request 1
Host: lab-id.web-security-academy.net
        │
        ▼
Front-end Validasi Host
        │
        ▼
Connection Ditandai Trusted
        │
        ▼
Request 2
Host: 192.168.0.1
        │
        ▼
Front-end Tidak Memvalidasi Lagi
        │
        ▼
Backend Menerima Request
        │
        ▼
Internal Admin Panel Terakses
```

---

# 🧠 Root Cause

Bug terjadi karena:

Front-end menyimpan status "Trusted Connection".

Yang divalidasi adalah:

```
Connection
```

Bukan

```
Request
```

Akibatnya request kedua mewarisi status request pertama.

---

# 🎯 Alur Eksploitasi

1.

Kirim request normal

```
GET /
Host: YOUR-LAB-ID
```

↓

Front-end menganggap koneksi valid.

2.

Dalam koneksi TCP yang sama kirim

```
GET /admin
Host: 192.168.0.1
```

↓

Admin Panel Internal terbuka.

3.

Salin request Delete User.

4.

Buat request

```
POST /admin/delete
Host: 192.168.0.1
```

5.

Kirim lagi menggunakan:

```
Send Group In Sequence
(Single Connection)
```

6.

User Carlos berhasil dihapus.

---

# 🔥 Kenapa Harus Dua Request?

Karena request pertama berfungsi sebagai:

```
Membangun Trusted Connection
```

Sedangkan request kedua:

```
Memanfaatkan Trusted Connection
```

Kalau langsung mengirim

```
Host: 192.168.0.1
```

Front-end belum memiliki state sehingga request ditolak.

---

# 🔥 Kenapa Harus Single Connection?

Karena bug terjadi pada:

```
TCP Connection
```

Bukan pada:

- Cookie
- Session
- Browser

Misalnya

```
Connection #1

GET /
Host: target.com

↓

Trusted
```

Kalau request berikutnya memakai:

```
Connection #2
```

State akan hilang.

---

# 🔥 Connection: keep-alive

Pada HTTP/1.1 koneksi persistent biasanya sudah default.

Karena itu pada beberapa lab header

```
Connection: keep-alive
```

sudah ada sejak awal.

Yang paling penting adalah Burp mengirim request menggunakan:

```
Send Group In Sequence
(Single Connection)
```

Header keep-alive hanya memastikan koneksi tidak ditutup.

---

# 🧠 TCP Singkat

TCP

```
Transmission Control Protocol
```

Karakteristik

✅ Reliable

✅ Paket tidak hilang

✅ Ada ACK

✅ Ada Retransmission

✅ Connection Oriented

HTTP berjalan di atas TCP.

Lab ini memanfaatkan kelemahan pada koneksi TCP tersebut.

---

# 🎯 Cara Berpikir Saat Hunting (Real Target)

Lab memberi tahu bug.

Di dunia nyata kita harus menemukan indikasinya sendiri.

Urutannya biasanya:

```
Host Header

↓

Perubahan Response

↓

Hipotesis

↓

Tes Berikutnya

↓

Analisis

↓

Eksploitasi
```

Jangan langsung memakai semua payload.

Biarkan response menentukan langkah berikutnya.

---

# 🔍 Observation 1

Host diubah

```
Host: attacker.com
```

↓

```
400 Invalid Host
```

## Arti

Server melakukan validasi Host.

Jangan berhenti.

Ini justru menarik.

---

## Yang Saya Lakukan

Pertama saya ingin tahu

SIAPA

yang mengembalikan response.

Apakah:

- Cloudflare
- Reverse Proxy
- Load Balancer
- Framework
- Backend

---

## Cara Membuktikannya

Lihat Header Response

Misalnya

```
Server: cloudflare
```

↓

Kemungkinan Cloudflare.

Atau

```
Server: nginx
```

↓

Kemungkinan Reverse Proxy.

Atau

```
Server: envoy
```

↓

Kemungkinan Envoy Proxy.

Atau

```
X-Powered-By:
Laravel
```

↓

Kemungkinan aplikasi.

---

# Kalau Cloudflare

Artinya request kemungkinan belum sampai backend.

Yang saya lakukan

Bukan lagi mencoba Host Header terus menerus.

Tetapi mencari:

- Origin IP
- Origin Domain
- Subdomain yang tidak diproteksi
- Endpoint lain

Karena validasi dilakukan sebelum request mencapai aplikasi.

---

# Kalau Reverse Proxy

Misalnya Nginx.

Saya mulai menguji parsing.

Contoh

```
Host: target.com.
```

```
Host: target.com:443
```

```
Host: TARGET.COM
```

```
Host: target.com:
```

Tujuannya

Melihat apakah parser membaca Host secara berbeda.

Kalau ada yang lolos

Berarti parsing tidak konsisten.

Saya lanjut:

- Host Header Injection
- Routing Based SSRF
- Cache Poisoning
- Connection State

---

# Kalau Framework

Misalnya Laravel.

Berarti request sudah sampai aplikasi.

Saya mulai cek fitur yang membangun URL.

Misalnya

- Password Reset
- Invite User
- Verify Email
- Canonical URL
- Redirect

Karena Host sering dipakai untuk membuat URL.

---

# 🔍 Observation 2

Host diubah

↓

```
302 Found
```

atau

```
Location:
https://attacker.com/
```

## Arti

Host dipakai aplikasi.

Yang saya lakukan

Saya cek:

- Forgot Password
- Email Verification
- Invite
- Redirect
- Canonical URL

Karena mungkin terjadi:

- Password Reset Poisoning
- Open Redirect
- Host Header Injection

---

# 🔍 Observation 3

Host diubah

↓

```
500 Internal Server Error
```

## Arti

Backend crash saat parsing Host.

Yang saya lakukan

Saya bandingkan

Host

```
localhost
```

```
127.0.0.1
```

```
target.com.
```

```
target.com:443
```

Kalau error berubah

Berarti parsing backend berbeda.

---

# 🔍 Observation 4

Host diubah

↓

Response sama persis.

Status

Body

Header

Content-Length

Sama.

## Arti

Kemungkinan Host tidak dipakai.

Saya tidak menghabiskan waktu lama.

Pindah ke teknik lain.

---

# 🧠 Proxy Behavior

Kadang saya ingin tahu

Apakah request diproses proxy.

Yang saya lakukan misalnya

Mengirim

```
OPTIONS *
```

atau

Absolute URL

```
GET http://example.com/
```

atau

Melihat Header

```
Via
```

```
X-Forwarded-For
```

```
X-Cache
```

```
CF-Ray
```

Tujuannya

Mengidentifikasi siapa yang memproses request.

Bukan langsung mencari bug.

---

# ☁️ CDN Misconfiguration

Kalau ternyata memakai Cloudflare.

Saya tidak langsung menyerah.

Saya mulai mencari

Origin Server.

Misalnya lewat

- DNS History
- Certificate Transparency
- Subdomain Lama
- IP Lama
- Shodan
- Censys

Kalau menemukan Origin IP.

Misalnya

```
203.0.113.20
```

Saya tidak mengubah Host.

Yang saya ubah adalah

DESTINATION

Di Burp Repeater.

Misalnya

Semula

```
https://target.com
```

Saya ganti menjadi

```
https://203.0.113.20
```

Tetapi Header tetap

```
Host: target.com
```

Kenapa?

Karena origin biasanya melayani Virtual Host berdasarkan Header Host.

Kalau memakai

```
Host: 203.0.113.20
```

Bisa jadi mendapat

404

atau

Default Website.

---

# Yang Dilihat Setelah Sampai Origin

Saya membandingkan

Apakah response berubah.

Misalnya

Normal

```
403
```

Origin

```
200
```

Atau

Normal

```
404
```

Origin

```
Admin Panel
```

Atau

Header berbeda.

Server berbeda.

Cookie berbeda.

Endpoint baru.

Semua itu menunjukkan request benar-benar sudah melewati CDN.

---

# 📚 Pelajaran Lab

- Connection State terjadi pada TCP Connection.
- Validasi dilakukan sekali.
- Request kedua mewarisi Trusted State.
- Single Connection jauh lebih penting daripada Cookie.
- Host Header bisa memengaruhi routing.
- Saat hunting jangan hanya melihat status code.
- Selalu tanyakan:
  - Siapa yang mengembalikan response?
  - Apa yang berubah?
  - Kenapa berubah?
  - Request berikutnya apa?

---

# 🎯 Pola Pikir Hunter

```
Observation

↓

Hipotesis

↓

Buktikan Dengan Request Baru

↓

Bandingkan Response

↓

Interpretasi

↓

Lanjut / Stop
```

Jangan pernah berhenti hanya karena melihat

```
400
```

atau

```
500
```

Karena response tersebut justru memberi petunjuk tentang bagaimana aplikasi memproses request.

---

# ⚡ Hafalan Cepat

```
Host Diubah

↓

Response Berubah

↓

Cari Siapa Yang Menjawab

↓

CDN?
Proxy?
Framework?

↓

Tes Parsing

↓

Bandingkan Response

↓

Eksploitasi
```

---

# 🔑 Inti Lab

**Bug Class**

```
Routing-Based SSRF
(Connection State Attack)
```

**Root Cause**

```
Front-end memvalidasi Host hanya pada request pertama dalam satu TCP Connection.
```

**Syarat Eksploitasi**

```
Single TCP Connection
```

**Target Internal**

```
192.168.0.1/admin
```

**Teknik Burp**

```
Send Group In Sequence
(Single Connection)
```

**Pola Hunting**

```
Observation
↓

Hypothesis
↓

Proof
↓

Interpretation
↓

Next Action
```
