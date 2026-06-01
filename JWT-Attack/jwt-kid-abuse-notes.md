# KID Abuse Notes (Extended)

## Apa itu KID?

`kid` = Key ID

Normal:

```text
kid
↓
server cari key
↓
verify JWT
```

Kalau lookup salah:

```text
↓
KID abuse
```

---

# Jenis-Jenis KID Abuse

## 1. KID Path Traversal

Fokus:

> baca file via traversal

Contoh:

```json
{
 "kid":"../../../../dev/null"
}
```

---

## 2. KID SQL Injection

Fokus:

> manipulasi query SQL

Contoh:

```json
{
 "kid":"' OR 1=1--"
}
```

---

## 3. KID NoSQL Injection

Fokus:

> manipulasi query NoSQL

Contoh:

```json
{
 "kid":{"$ne":null}
}
```

---

## 4. KID Null / Default Key

Fokus:

> paksa server pakai key default

Contoh:

```json
{
 "kid":""
}
```

atau:

```json
{
 "kid":null
}
```

---

## 5. KID Cache Collision

Fokus:

> tabrakan cache lookup

Contoh server jelek:

```python
cache_key = kid[:4]
```

Normal:

```json
{
 "kid":"admin-secret"
}
```

Server:

```text
cache:
admin -> secret_admin
```

Attacker:

```json
{
 "kid":"admin-fake"
}
```

Server:

```text
kid[:4]
↓
admin
↓
ambil:
secret_admin 😭
```

Akibat:

```text
lookup salah
↓
key salah dipakai
```

Inti:

> server ambil key cache yang salah

---

## 6. KID Arbitrary File Read

Fokus:

> baca file internal

Contoh:

```json
{
 "kid":"config/prod.key"
}
```

Beda dengan traversal:

Traversal:

```text
../../
```

Arbitrary read:

```text
langsung file internal
```

---

# Ringkasan

| Jenis | Fokus | Target |
|---|---|---|
| Path traversal | Traversal | File |
| SQLi | SQL | Database |
| NoSQLi | NoSQL | Database |
| Null/default | Fallback | Default key |
| Cache collision | Cache | Cached key |
| Arbitrary file read | File read | Internal file |

---

## Hafalan

```text
KID abuse
↓
server salah lookup key
↓
bisa:
file
database
cache
default
```
