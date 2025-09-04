
# Rating Rules JSON

## 1. `rating_rules.json`

`rating_rules.json` adalah **konfigurasi parser rating**.
Isinya aturan berbasis regex untuk mendeteksi pola rating di teks review (contoh: `5/5`, `⭐⭐⭐⭐⭐`, `bintang 1000++`).

Parser ini mengubah **teks bebas** ➝ **angka rating 1.0–5.0**.
---

## 2. Struktur File

```json
{
  "schema_version": "1.0",
  "defaults": {
    "flags": ["IGNORECASE", "MULTILINE"],
    "min_rating": 1.0,
    "max_rating": 5.0
  },
  "rules": [
    {
      "id": "...",
      "priority": ...,
      "type": "...",
      "pattern": "...",
      "example": "..."
    }
  ],
  "blacklist": [...]
}
```

### Penjelasan bagian:

* **`schema_version`** → versi schema (saat ini `1.0`).
* **`defaults`** → aturan global:

  * `flags`: opsi regex default.
  * `min_rating` / `max_rating`: clamp rating ke rentang 1–5.
* **`rules`** → kumpulan aturan rating.
* **`blacklist`** → daftar pola yang diabaikan agar tidak jadi false positive.

---

## 3. Tipe Rule

* **`extract`** → ambil angka dari regex group.

  > contoh: `([1-5])\s*bintang` → `"5 bintang"` ➝ `5.0`

* **`assign`** → jika cocok, langsung tetapkan nilai.

  > contoh: `"bintang 1000++"` ➝ `5.0`

* **`count_emoji`** → hitung jumlah emoji ⭐/★/🌟.

  > contoh: `"⭐⭐⭐"` ➝ `3.0`

---

## 4. Priority

**Priority** = urutan eksekusi rule.

* Angka lebih tinggi ➝ diproses lebih dulu.
* Berguna kalau ada lebih dari satu pola di teks.

Contoh:

```
Barang oke ⭐⭐⭐⭐⭐ (4.5/5)
```

* `stars_repeated` ➝ 5
* `frac_out_of_5` ➝ 4.5 (lebih tinggi priority)

➡️ Hasil akhir: **4.5**

---

## 5. Clamp

**Clamp** = paksa nilai ke rentang tertentu (`min_rating`, `max_rating`).

* `"bintang 10"` ➝ ekstrak 10 ➝ clamp ➝ **5.0**
* `"bintang 0"` ➝ ekstrak 0 ➝ clamp ➝ **1.0**

---

## 6. Contoh Aturan

### Angka /5

```json
{
  "id": "frac_out_of_5",
  "priority": 100,
  "type": "extract",
  "pattern": "\\b([1-5](?:[.,]5)?)\\s*/\\s*5\\b",
  "value_group": 1,
  "postprocess": {"replace": {",": "."}},
  "clamp": [1.0, 5.0],
  "example": "4.5/5 mantap"
}
```

Hasil: `"4,5/5"` ➝ `4.5`.

---

### Bintang Banyak

```json
{
  "id": "bintang_many",
  "priority": 95,
  "type": "assign",
  "pattern": "\\bbintang\\s*\\d{3,}(?:\\+{1,2})?(?=\\s|$|[^\\w])",
  "value": 5.0,
  "example": "bintang 1000++"
}
```

Hasil: langsung **5.0**.

---

### Emoji Bintang

```json
{
  "id": "stars_repeated",
  "priority": 80,
  "type": "count_emoji",
  "emojis": ["⭐", "★", "🌟"],
  "min_count": 2,
  "max_count": 5,
  "example": "⭐⭐⭐⭐⭐"
}
```

Hasil: hitung emoji, clamp ke max 5.

---

## 7. Cara Pakai

### Load Config

```python
import json
with open("rating_rules.json", encoding="utf-8") as f:
    CFG = json.load(f)
```

### Ekstrak Rating (Sesuaikan lagi lebih lanjut)

```python
import re
FLAGS = re.IGNORECASE | re.MULTILINE

def extract_rating(text: str, cfg: dict):
    for r in sorted(cfg["rules"], key=lambda x: -x["priority"]):
        rx = re.compile(r["pattern"], FLAGS)
        if r["type"] == "assign":
            if rx.search(text):
                return float(r.get("value", 5.0))
        elif r["type"] == "extract":
            m = rx.search(text)
            if not m: continue
            val = m.group(int(r["value_group"]))
            for a,b in r.get("postprocess", {}).get("replace", {}).items():
                val = val.replace(a, b)
            score = float(val) + float(r.get("add", 0.0))
            lo, hi = cfg["defaults"]["min_rating"], cfg["defaults"]["max_rating"]
            return max(lo, min(hi, score))
        elif r["type"] == "count_emoji":
            cnt = sum(text.count(e) for e in r["emojis"])
            if cnt >= r["min_count"]:
                return float(min(cnt, r["max_count"]))
    return None
```


mau aku bikinin juga contoh visual diagram alur parser (dari teks ➝ regex match ➝ rating final) biar gampang ditaruh di README.md?
