# WebAcademy

[![PortSwigger Web Security Academy](https://img.shields.io/badge/PortSwigger-Web%20Security%20Academy-FF6633?logo=portswigger&logoColor=white)](https://portswigger.net/web-security)
[![License: MIT](https://img.shields.io/badge/Lisensi-MIT-blue.svg)](LICENSE)

Arsip catatan belajar dan writeup teknis dari **PortSwigger Web Security Academy**. Setiap catatan mendokumentasikan akar masalah, alasan di balik eksploitasi, payload yang dipakai, dan cara memperbaikinya.

Saat ini berisi **31 lab** yang tersebar dalam **8 kategori**, dengan fokus pada autentikasi, kontrol akses, keamanan API, dan mekanisme caching.

---

## Tentang

Saya **Rijwan Maulana** ([@siiwannn](https://github.com/siiwannn)). Repository ini mencatat proses belajar saya di bidang keamanan aplikasi web, mulai dari memahami konsep hingga membuktikan dampak eksploitasi dan merumuskan perbaikan.

Setiap catatan disusun agar bisa dibaca ulang tanpa perlu membuka lab-nya lagi: alasan di balik setiap langkah lebih penting daripada urutan klik.

---

## Fokus Pembelajaran

- Keamanan autentikasi dan sesi — JWT, OAuth, alur reset password
- Kontrol akses dan celah logika bisnis
- Server-Side Request Forgery (SSRF) dan serangan host header
- Web cache poisoning dan web cache deception
- Keamanan API — mass assignment, server-side parameter pollution, endpoint tersembunyi
- Celah API GraphQL
- Injeksi dan validasi input

---

## Struktur

```text
WebAcademy/
├── API-TESTING/            # Parameter pollution, mass assignment, penemuan endpoint
├── Business-Logic/         # Celah logika dan bypass autentikasi
├── GraphQL/                # Query GraphQL dan kebocoran field
├── Host-Header/            # Serangan host header, cache poisoning, routing SSRF
├── JWT-Attack/             # Serangan algoritma, kunci, dan header JWT
├── OAuth/                  # Celah OAuth dan OpenID Connect
├── SSRF/                   # Dasar SSRF dan teknik bypass
└── Web-Cache-Deception/    # Teknik cache deception
```

---

## Indeks Catatan

<details>
<summary><strong>Serangan JWT</strong> — 8 lab</summary>

- [Unverified Signature](JWT-Attack/jwt-01-unverified-signature.md)
- [Bypass `alg:none`](JWT-Attack/jwt-02-alg-none.md)
- [Weak Signing Key](JWT-Attack/jwt-03-weak-signing-key.md)
- [JWK Injection](JWT-Attack/jwt-04-jwk-injection.md)
- [JKU Injection](JWT-Attack/jwt-05-jku-injection.md)
- [KID Path Traversal](JWT-Attack/jwt-06-kid-path-traversal.md)
- [Algorithm Confusion](JWT-Attack/jwt-07-algorithm-confusion.md)
- [Algorithm Confusion (No Exposed Key)](JWT-Attack/jwt-08-alg-confusion-no-exposed-key.md)

Catatan pendukung: [Ringkasan JWT](JWT-Attack/jwt-summary.md) · [Penyalahgunaan KID](JWT-Attack/jwt-kid-abuse-notes.md)

</details>

<details>
<summary><strong>Pengujian API</strong> — 5 lab</summary>

- [Server-Side Parameter Pollution pada Query String](API-TESTING/Exploiting-Server-Side-Parameter-Pollution-in-a-Query-String.md)
- [Server-Side Parameter Pollution pada REST URL](API-TESTING/Exploiting-Server-Side-Parameter-Pollution-in-a-REST-URL.md)
- [Mass Assignment Vulnerability](API-TESTING/Exploiting-a-Mass-Assignment-Vulnerability.md)
- [Eksploitasi Endpoint API melalui Dokumentasi](API-TESTING/Exploiting-an-API-Endpoint-Using-Documentation.md)
- [Menemukan dan Mengeksploitasi Endpoint API yang Tidak Terpakai](API-TESTING/Finding-and-Exploiting-an-Unused-API-Endpoint.md)

</details>

<details>
<summary><strong>Serangan HTTP Host Header</strong> — 7 lab</summary>

- [Authentication Bypass](Host-Header/Authentication-Bypass.md)
- [Host Validation Bypass via Connection State Attack](Host-Header/Host-Validation-Bypass-via-Connection-State-Attack.md)
- [Password Reset Poisoning](Host-Header/Password-Reset-Poisoning.md)
- [Password Reset Poisoning via Hanging Markup](Host-Header/Password-Reset-Poisoning-via-Hanging-Markup.md)
- [Routing-Based SSRF via Host Header](Host-Header/Routing-Based-SSRF-via-Host-Header.md)
- [SSRF via Flawed Request Parsing](Host-Header/SSRF-via-Flawed-Request-Parsing.md)
- [Web Cache Poisoning via Ambiguous Requests](Host-Header/Web-Cache-Poisoning-via-Ambiguous-Requests.md)

</details>

<details>
<summary><strong>Web Cache Deception</strong> — 5 lab</summary>

- [Cache Server Normalization](Web-Cache-Deception/Exploiting-Cache-Server-Normalization-for-Web-Cache-Deception.md)
- [Exact Match Cache Rules](Web-Cache-Deception/Exploiting-Exact-Match-Cache-Rules-for-Web-Cache-Deception.md)
- [Origin Server Normalization](Web-Cache-Deception/Exploiting-Origin-Server-Normalization-for-Web-Cache-Deception.md)
- [Path Delimiters](Web-Cache-Deception/Exploiting-Path-Delimiters-for-Web-Cache-Deception.md)
- [Path Mapping](Web-Cache-Deception/Exploiting-Path-Mapping-for-Web-Cache-Deception.md)

</details>

<details>
<summary><strong>Celah API GraphQL</strong> — 2 lab</summary>

- [Mengakses Postingan GraphQL Privat](GraphQL/Accessing-private-GraphQL-posts.md)
- [Kebocoran Field GraphQL Privat secara Tidak Sengaja](GraphQL/Accidental-exposure-of-private-GraphQL-fields.md)

</details>

<details>
<summary><strong>Server-Side Request Forgery (SSRF)</strong> — 2 lab</summary>

- [SSRF Dasar terhadap Server Lokal](SSRF/Basic-SSRF-against-the-Local-Server.md)
- [SSRF dengan Filter Input Berbasis Blacklist](SSRF/SSRF-with-blacklist-based-input-filter.md)

</details>

<details>
<summary><strong>Autentikasi OAuth</strong> — 1 lab</summary>

- [SSRF melalui OpenID Dynamic Client Registration](OAuth/SSRF-via-OpenID-Dynamic-Client-Registration.md)

</details>

<details>
<summary><strong>Celah Logika Bisnis</strong> — 1 lab</summary>

- [Bypass Autentikasi melalui Encryption Oracle](Business-Logic/Authentication-bypass-via-encryption-oracle.md)

</details>

---

## Cara Catatan Disusun

1. **Akar masalah** — kerentanan yang sebenarnya, bukan gejala yang terlihat.
2. **Proses penemuan** — observasi atau anomali yang mengarah ke celah tersebut.
3. **Eksploitasi** — payload, request, dan mekanisme di baliknya.
4. **Perbaikan** — kontrol defensif yang menghilangkan akar masalah.

Penekanan ada pada penalaran dan pola yang bisa dipakai ulang, bukan tangkapan layar langkah demi langkah.

---

## Tools

Burp Suite (Repeater, Intruder, Collaborator), Browser DevTools, tool JWT dan wordlist kustom, serta query introspeksi GraphQL — semuanya dijalankan di lingkungan lab PortSwigger Web Security Academy.

---

## Rencana Selanjutnya

- [ ] Menyelesaikan seri lab SQL Injection dan XSS
- [ ] Menambahkan writeup access control dan autentikasi
- [ ] Mendokumentasikan lab file upload dan path traversal
- [ ] Menambahkan catatan HTTP request smuggling dan deserialization
- [ ] Menerbitkan checklist metodologi pengujian yang ringkas

---

## Lisensi

Dirilis di bawah [MIT License](LICENSE).

© 2026 Rijwan Maulana
