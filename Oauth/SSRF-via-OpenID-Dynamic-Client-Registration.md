# SSRF via OpenID Dynamic Client Registration

## Konsep Inti

Lab ini menunjukkan SSRF melalui **OAuth/OpenID Dynamic Client Registration**.

Intinya:

    Attacker-controlled URL
            ↓
       OAuth Server
            ↓
    Server-side HTTP request
            ↓
    Attacker / internal resource

Yang menyebabkan SSRF bukan `redirect_uris`, tetapi `logo_uri`.

### OAuth 2.0 vs OpenID Connect
* **OAuth 2.0** = authorization.
* **OIDC (OpenID Connect)** = identity/authentication layer di atas OAuth 2.0.

    OAuth 2.0 + Identity / Authentication = OIDC

`scope=openid` adalah indikator kuat bahwa flow menggunakan OIDC. Kalau `scope=openid` tidak ada, jangan langsung di-*skip*. Flow tersebut masih bisa merupakan OAuth 2.0. 

Contoh: `scope=read` → tetap bisa OAuth 2.0.

Jadi:
* `scope=openid` ada → kemungkinan OIDC
* `scope=openid` tidak ada → masih bisa OAuth 2.0

> **Note:** OAuth 2.0 dan OIDC bukan vulnerability.

---

## Ciri OAuth/OIDC Saat Hunting

**Parameter yang sering muncul:**
* `client_id`
* `redirect_uri`
* `response_type`
* `scope`
* `state`
* `nonce`
* `code`
* `code_challenge`
* `code_challenge_method`

**Endpoint yang sering ditemukan:**
* `/authorize`
* `/token`
* `/userinfo`
* `/oauth/`
* `/oauth2/`
* `/oidc/`
* `/callback`
* `/sso/`

Kalau menemukan request seperti ini:
    GET /authorize?client_id=xxx&redirect_uri=https://target.com/callback&response_type=code&scope=read

Jangan langsung di-skip hanya karena tidak ada `scope=openid`.

---

## .well-known/openid-configuration

Endpoint `/.well-known/openid-configuration` adalah *discovery endpoint* OIDC.
Tidak semua target pasti memiliki endpoint ini. Tergantung pada:
* Penggunaan OIDC
* Identity Provider
* OAuth/OIDC implementation
* Konfigurasi Authorization Server

Jadi jangan berpikir setiap target pasti ada di `target.com/.well-known/openid-configuration`. 

Yang benar:
    Temukan OAuth/OIDC flow
            ↓
    Identifikasi Authorization Server
            ↓
    Coba discovery pada server tersebut

Contoh jika `target.com` me-redirect (302) ke `auth.target.com/authorize`, maka lebih masuk akal memeriksa:
`auth.target.com/.well-known/openid-configuration`

Selain itu, OAuth 2.0 Authorization Server Metadata dapat menggunakan:
`/.well-known/oauth-authorization-server`

---

## Dynamic Client Registration

Dynamic Client Registration memungkinkan client mendaftarkan dirinya ke OAuth server.

Contoh Request:
    POST /reg HTTP/1.1
    Content-Type: application/json
    
    {
        "redirect_uris": [
            "https://example.com"
        ]
    }

Server kemudian akan memberikan `client_id`.

Endpoint-nya tidak harus bernama `/reg`. Saat real hunting, cari konsep:
* `/reg`
* `/register`
* `/registration`
* `/oauth/register`
* `/oauth2/register`

atau lihat `registration_endpoint` pada *discovery document*.

---

## redirect_uris vs SSRF

`redirect_uris` digunakan untuk menentukan callback OAuth.

    {
        "redirect_uris": [
            "https://target.com/callback"
        ]
    }

**Flow:**
    User → Target → OAuth Server → Login / Authorization → redirect_uri → Target callback

Jadi `redirect_uri` berkaitan dengan browser redirect.

### Kenapa redirect_uri Bukan SSRF?
Karena yang melakukan request biasanya adalah **browser**.
Sedangkan SSRF adalah **Server-side** HTTP request.

**Mental model penting:**
Tanyakan siapa yang mengakses URL tersebut: browser atau server?
* **Browser** → redirect-related vulnerability.
* **Server** → SSRF investigation.

### redirect_uri dan Open Redirect
`redirect_uri` yang bisa dimanipulasi tidak otomatis berarti Open Redirect.
Jika server merespons `400 Invalid redirect_uri`, maka tidak ada *open redirect* dari *behavior* tersebut.
Namun, jika server melakukan:

    HTTP/1.1 302 Found
    Location: https://evil.example

→ Ada indikasi attacker-controlled redirect.

Tetap evaluasi:
* Apakah `redirect_uri` memang tidak divalidasi?
* Apakah `client_id` milik attacker atau korban?
* Apakah authorization code/token ikut terkirim?
* Apakah ada impact terhadap akun korban?

---

## logo_uri — Sumber SSRF Lab

Berbeda dengan `redirect_uri`, `logo_uri` memberi tahu OAuth server lokasi logo client.

Contoh payload:

    {
        "redirect_uris": [
            "https://example.com"
        ],
        "logo_uri": "https://BURP-COLLABORATOR"
    }

Kemudian akses endpoint: `GET /client/<CLIENT-ID>/logo` untuk meminta OAuth server mengambil logo tersebut.

**Alurnya:**
    logo_uri → OAuth Backend → HTTP Fetch → BURP Collaborator

Karena request dilakukan oleh *backend*, ini merupakan **SSRF primitive**.

### Trigger vs Sink
* **Input:** `logo_uri` (Attacker mengontrol URL)
* **Sink:** Backend melakukan `HTTP GET <logo_uri>`
* **Trigger:** `GET /client/<CLIENT-ID>/logo`

---

## Kenapa Pakai Burp Collaborator?

Collaborator (OAST) digunakan untuk membuktikan bahwa server benar-benar melakukan request. Jika Collaborator menerima HTTP interaction, berarti SSRF primitive berhasil dikonfirmasi.

---

## Lab Flow

1. Login sebagai `wiener:peter`
2. Buka discovery endpoint: `/.well-known/openid-configuration`
3. Temukan `registration_endpoint` (misal: `/reg`)
4. Register client:
   
        POST /reg
        Content-Type: application/json
        
        {
            "redirect_uris": ["https://example.com"],
            "logo_uri": "https://BURP-COLLABORATOR"
        }
   
5. Dapatkan `client_id` baru.
6. Trigger SSRF: `GET /client/<CLIENT-ID>/logo`
7. Cek Collaborator. Jika ada HTTP interaction → SSRF confirmed.
8. Eksploitasi, arahkan `logo_uri` ke:
   `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/`
9. Trigger endpoint logo lagi. Response metadata cloud akan dikembalikan.

> Lab ini menunjukkan **Response-Based SSRF** di mana response target dikembalikan kepada attacker.

---

## Real Hunting Strategy

Jangan hanya mencari `POST /reg`. Cari mekanismenya:
    Login / SSO → Intercept OAuth traffic → Identifikasi Authorization Server → Discovery → registration_endpoint → Dynamic Client Registration

Kalau registration endpoint tersedia, lihat property yang dapat dikontrol client (Contoh: `logo_uri`, `jwks_uri`, `policy_uri`, `tos_uri`, `request_uri`). 
**Pertanyaan utamanya:** Apakah backend melakukan request ke URL tersebut?

### SSRF Testing di Real World
Gunakan domain yang dikontrol sendiri untuk melakukan OAST/Collaborator testing. Jika dapat hit, SSRF primitive confirmed.

**Kalau Private IP / AWS Metadata Diblok:**
* Collaborator ✓
* External URL ✓
* 127.0.0.1 ✗
* 169.254.169.254 ✗

Jangan langsung menyimpulkan SSRF tidak ada. Ini berarti:
`SSRF primitive + Destination filtering / egress filtering`

Server memang melakukan server-side request, tetapi membatasi *destination* tertentu. SSRF tidak harus berujung ke AWS metadata, bisa juga ke:
* Internal API
* Localhost service
* Internal admin panel
* Monitoring service

**Mental Model:**
    SSRF primitive → Reachability → Apa yang dapat dijangkau? → Apakah response dapat dibaca? → Apa security impact-nya?

---

## Observation → Interpretation → Test → Evidence

Gunakan pola ini saat real hunting:

1. **Observation:** Collaborator menerima request.
2. **Interpretation:** Backend melakukan server-side HTTP request.
3. **Hypothesis:** Terdapat SSRF primitive.
4. **Test:** Gunakan controlled endpoint dan evaluasi (HTTP method, headers, source, response reflection, redirect behavior, destination restrictions).
5. **Evidence:** Collaborator interaction berasal dari infrastructure target.
6. **Next:** Evaluasi reachability dan security impact.

---

## 🧠 Hafalan Cepat

* **OAuth 2.0** → Authorization
* **OIDC** → OAuth 2.0 + Identity
* `scope=openid` → Indikator OIDC
* `redirect_uri` → Browser redirect → Redirect-related testing
* `logo_uri` (URL di-fetch backend) → Server-side request → **SSRF**
* **Collaborator hit** → SSRF primitive confirmed
* **Private IP blocked** → Bukan berarti SSRF tidak ada, kemungkinan *destination filtering*

**Inti:** Jangan fokus pada nama endpoint atau parameter. Fokus pada siapa yang memproses URL tersebut.
* **Browser** → redirect
* **Server** → SSRF