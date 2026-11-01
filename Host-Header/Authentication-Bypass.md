# Authentication bypass - Host Header Attack

## 1. Ringkasan Lab

**Nama Lab:** Authentication bypass via Host header manipulation  
**Kategori:** HTTP Host Header Injection  
**Tujuan:**  
- Akses panel admin  
- Menghapus user carlos

**Konsep utama:**  
Server mempercayai nilai `Host` yang dikirim client sehingga request dianggap berasal dari localhost (trusted host).

---

## 2. Langkah Eksploitasi (Lab)

1. Request normal ke `/` → 200 OK
2. Cek `/robots.txt` → ditemukan `/admin`
3. Akses `/admin` → 403 Forbidden
4. Uji di Burp Repeater:

```http
GET /admin HTTP/1.1
Host: localhost
```

5. Response berubah → akses admin berhasil
6. Delete user:

```http
GET /admin/delete?username=carlos
Host: localhost
```

7. Lab solved

---

## 3. Kenapa `Host: localhost` bisa berhasil?

Karena backend mempercayai nilai `Host`.

Alurnya:

```
Client
    │
    ▼
Host: localhost
    │
    ▼
Server menganggap request berasal dari localhost
    │
    ▼
Authentication / Access Control dilewati
    │
    ▼
Admin panel dapat diakses
    │
    ▼
Delete user carlos
```

Jadi bug utamanya adalah **Host Header Injection**.

Sedangkan **Authentication Bypass hanyalah impact** dari bug tersebut.

---

## 4. Lab vs Real Case

### Lab

- Endpoint admin sudah disediakan
- Impact dibuat sederhana agar mudah dipahami
- Bug dibuktikan dengan bypass admin

### Real Case

Jarang menemukan kasus yang persis sama.

Host Header lebih sering berdampak pada:

- Password Reset Poisoning
- Email Link Generation
- Open Redirect
- Web Cache Poisoning
- Routing-based SSRF
- Virtual Host Routing

Authentication bypass tetap mungkin terjadi, tetapi jauh lebih jarang dibanding contoh di atas.

---

## 5. Apakah Host relevan setelah IDOR gagal?

Tidak otomatis.

403 pada IDOR berarti:

- authorization bekerja
- belum tentu ada Host Header bug

Host baru layak diuji jika:

- aplikasi menggunakan Host dalam logic
- response berubah saat Host diganti
- routing berubah
- redirect berubah
- absolute URL berubah

---

## 6. Cara Hunting Host Header Bug

Cari indikasi seperti:

- response berubah saat Host diganti
- redirect berubah
- email link berubah
- cache response berubah
- virtual host berubah
- absolute URL berubah

Jika ada perubahan, lanjutkan investigasi.

Jika tidak ada perubahan, lanjut ke hipotesis lain.

---

## 7. FAQ (Hal yang Sering Membingungkan)

### Q: Kenapa nama lab Authentication Bypass padahal topiknya Host Header?

**Jawaban**

Karena PortSwigger memberi nama lab berdasarkan **impact**.

Bug sebenarnya tetap **HTTP Host Header Injection**.

Authentication Bypass hanyalah akibat dari bug tersebut.

---

### Q: Berarti Host Header Injection selalu digunakan untuk bypass admin?

**Jawaban**

Tidak.

Bypass admin hanyalah salah satu contoh impact.

Di aplikasi lain impact bisa berupa:

- Password Reset Poisoning
- Cache Poisoning
- Open Redirect
- Routing-based SSRF

---

### Q: Kalau menemukan endpoint DELETE atau endpoint lain, boleh mencoba mengganti Host?

**Jawaban**

Boleh.

Itu merupakan salah satu variasi request yang wajar saat melakukan pengujian pada target yang memang mengizinkan testing (Lab / Bug Bounty Scope).

Jika response berubah, kemungkinan ada sesuatu yang menarik.

Jika tidak berubah, lanjutkan ke teknik lain.

---

### Q: Jadi kenapa di lab cukup ganti Host menjadi localhost lalu bisa delete carlos?

**Jawaban**

Karena server salah mempercayai nilai `Host`.

Server menganggap request berasal dari localhost sehingga authentication atau access control dilewati.

Akibatnya endpoint admin bisa diakses dan user carlos dapat dihapus.

---

## 8. Mindset Hunting

Jangan berpikir:

> Host Header = bypass admin

Tetapi berpikirlah:

> Apakah aplikasi menggunakan nilai Host untuk mengambil keputusan?

Jika ya, Host Header Injection layak diuji.

Impact bisa berbeda pada setiap aplikasi.

---

## 9. Kesimpulan

- Bug = HTTP Host Header Injection
- Penyebab = Backend mempercayai Host dari client
- Impact lab = Authentication Bypass → akses admin → delete user carlos
- Real case belum tentu bypass admin
- Fokus hunting adalah mencari fitur yang menggunakan Host, bukan mencari endpoint admin
