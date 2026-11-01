<a id="readme-top"></a>

<div align="center">

<h1>WebAcademy</h1>

<p>Catatan belajar dan writeup teknis dari <strong>PortSwigger Web Security Academy</strong>.<br />
Dokumentasi akar masalah, proses eksploitasi, payload, dan rekomendasi perbaikan.</p>

[![PortSwigger Web Security Academy][portswigger-shield]][portswigger-url]
[![License][license-shield]][license-url]

</div>

---

<details>
<summary>Daftar Isi</summary>
<ol>
<li><a href="#ikhtisar">Ikhtisar</a></li>
<li><a href="#fokus-pembelajaran">Fokus Pembelajaran</a></li>
<li><a href="#struktur">Struktur</a></li>
<li><a href="#indeks-catatan">Indeks Catatan</a></li>
<li><a href="#metodologi">Metodologi</a></li>
<li><a href="#tools">Tools</a></li>
<li><a href="#referensi">Referensi</a></li>
<li><a href="#lisensi">Lisensi</a></li>
</ol>
</details>

---

## Ikhtisar

Repository ini berisi **31 lab** dari **8 kategori** PortSwigger Web Security Academy, dengan penekanan pada autentikasi, kontrol akses, keamanan API, dan mekanisme caching.

Setiap catatan disusun agar dapat dipelajari kembali tanpa membuka lab aslinya. Fokusnya bukan urutan langkah, tetapi alasan di balik setiap keputusan: mengapa celah itu ada, bagaimana dampaknya dibuktikan, dan kontrol apa yang seharusnya mencegahnya.

<p align="right">(<a href="#readme-top">kembali ke atas</a>)</p>

## Fokus Pembelajaran

- **Autentikasi dan sesi** — JWT, OAuth, serta alur reset password
- **Kontrol akses** — otorisasi, IDOR, dan celah logika bisnis
- **Server-side** — SSRF, host header attacks, dan cache poisoning
- **Keamanan API** — mass assignment, parameter pollution, dan endpoint tersembunyi
- **GraphQL** — query introspection dan kebocoran field
- **Injeksi** — validasi input dan filter bypass

<p align="right">(<a href="#readme-top">kembali ke atas</a>)</p>

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

Setiap direktori berisi writeup dengan struktur seragam: tujuan lab, analisis, eksploitasi, payload, akar masalah, dan perbaikan.

<p align="right">(<a href="#readme-top">kembali ke atas</a>)</p>

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

<p align="right">(<a href="#readme-top">kembali ke atas</a>)</p>

## Metodologi

Setiap writeup mengikuti alur yang sama agar mudah dibandingkan dan dipelajari kembali:

1. **Identifikasi** — tentukan titik input dan perilaku aplikasi yang mencurigakan.
2. **Analisis** — pahami akar masalah dan asumsi yang salah di sisi server.
3. **Eksploitasi** — susun payload, buktikan dampaknya, dan catat responsnya.
4. **Perbaikan** — rumuskan kontrol defensif yang menghilangkan akar masalah.

Penekanan ada pada penalaran dan pola yang dapat dipakai ulang, bukan tangkapan layar langkah demi langkah.

<p align="right">(<a href="#readme-top">kembali ke atas</a>)</p>

## Tools

- **Burp Suite** — Repeater, Intruder, dan Collaborator
- **Browser DevTools** — inspeksi request dan penyimpanan sisi klien
- **JWT tooling** — decoding, signing, dan wordlist kustom
- **GraphQL** — query introspeksi dan eksplorasi schema

<p align="right">(<a href="#readme-top">kembali ke atas</a>)</p>

## Referensi

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [Web Security Academy Learning Paths](https://portswigger.net/web-security/learning-paths)
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)

<p align="right">(<a href="#readme-top">kembali ke atas</a>)</p>

## Lisensi

Dirilis di bawah [MIT License](LICENSE).

© 2026 Rijwan Maulana

<p align="right">(<a href="#readme-top">kembali ke atas</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->

[portswigger-shield]: https://img.shields.io/badge/PortSwigger-Web%20Security%20Academy-FF6633?style=for-the-badge&logo=portswigger&logoColor=white
[portswigger-url]: https://portswigger.net/web-security
[license-shield]: https://img.shields.io/badge/Lisensi-MIT-2ea44f?style=for-the-badge
[license-url]: LICENSE
