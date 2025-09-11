# Regex Notes - Indonesian PII (KTP/NIK & Address)

## 1. KTP / IDs

**Regex**

`(?<!\d)(1[1-9]|21|[37][1-6]|5[1-3]|6[1-5]|[89][12])\d{2}\d{2}([04][1-9]|[1256][0-9]|[37][01])(0[1-9]|1[0-2])\d{2}\d{4}(?!\d)`

**Penjelasan**

* `(?<!\d)` dan `(?!\d)` memastikan tidak ada digit lain di kiri/kanan → menghindari overmatch.
* 2 digit provinsi: `1[1-9]|21|[37][1-6]|5[1-3]|6[1-5]|[89][12]` → range kode provinsi Indonesia.
* Lanjut 2+2 digit kode kabupaten/kecamatan.
* Tanggal lahir: `(01–31)` untuk laki-laki, `(41–71)` untuk perempuan (tanggal + 40).
* Bulan lahir: `01–12`.
* Tahun lahir: 2 digit.
* Sequence: 4 digit.

**Best Practice**

* Jalankan **pre-clean digit-run**: hapus spasi/titik/strip antar digit sebelum regex (contoh `"3276 1207 0501 0003"` → `"3276120705010003"`).
* Gunakan replacement `[ID]` untuk masking.

**Contoh**

* ✅ `3276120705010003` → match
* ❌ `3276 1207 0501 0003` → tidak match (sebelum pre-clean)


## 2. Address (Alamat Indonesia)

### 2.1 Tujuan

Mendeteksi alamat nyata dalam teks e-commerce (deskripsi/ulasan), lalu **mask** menjadi `[ADDR]`.

### 2.2 Komponen Regex

* **Street**: `Jl./Jalan/Gg/Gang + nama jalan`
  *(Tidak dihitung sebagai komponen admin untuk validasi.)*
* **House Number**: `No/Nomor/Nomer/Nmr/# + isi (angka/huruf)`
* **RT/RW**: `RT xx RW yy` (keduanya wajib angka)
* **Kelurahan**: `Kel/ Kelurahan + nama`
* **Kecamatan**: `Kec/ Kecamatan + nama`
* **Kabupaten/Kota**: `Kab/Kabupaten/Kota + nama`
* **Provinsi**: `Prov/ Provinsi + nama`
* **ZIP (Kode Pos)**: angka 5 digit
* **PO BOX**: `Kotak Pos/ K.P./ KP + angka`

### 2.3 Trigger Window

```
\b(Jl\.?|Jalan|Gg\.?|Gang|RT|RW|Kel(?:urahan)?\.?|Kec(?:amatan)?\.?|Kab(?:upaten)?\.?|Kota|Prov(?:insi)?\.?|K\.?\s?P\.?|KP)\b
```

> **Catatan**: **ZIP tidak dimasukkan** ke trigger. Angka 5 digit sering muncul sebagai harga/SKU, jadi ZIP hanya divalidasi **setelah** window terbentuk.

### 2.4 Aturan Validasi (Token Proximity)

Alamat dianggap **valid** jika:

1. **PO BOX** terdeteksi → auto valid.
2. Minimal **2 komponen admin** (dari: house, rtrw, kel, kec, kabkota, prov, zip) **berdekatan (≤ N token)**.
3. **ZIP** wajib ditemani **salah satu** dari **Kel/Kec/Kab/Kota/Prov** → zip sendirian dianggap noise.
4. **Tolak label kosong**:

   * `RT/RW` tanpa angka
   * `Kelurahan/Kecamatan/Kabupaten/Kota/Provinsi` tanpa nama setelahnya

### 2.5 Hubungan Trigger & Komponen (Cara Pakai)

* **Trigger** = alarm awal untuk menandai **calon alamat** di teks, lalu **ambil window** kecil di sekitarnya.
* **Komponen** = potongan detail alamat yang dideteksi **di dalam window** (house, rtrw, kel, dsb).
* **Validator** = logika yang memutuskan **valid/tidak** berdasarkan kombinasi komponen:

  * PO BOX → valid
  * ≥2 komponen admin dan **dekat** (≤ N token) → valid
  * ZIP tanpa konteks admin area → tidak valid
  * Label kosong → tidak valid

> **Masking terjadi per-window**: jika sebuah window valid, **seluruh span alamat** pada window itu di-replace jadi `[ADDR]` (bukan per komponen).

### 2.6 Rekomendasi Konfigurasi

* `MAX_TOKEN_GAP = 6` → balanced
* `MAX_TOKEN_GAP = 4` → strict
* `MAX_TOKEN_GAP = 8` → recall (lebih longgar)

### 2.7 Contoh Valid

* `Jl. Teuku Umar No 76 Denpasar 80113`
* `Kelurahan Melati, Kecamatan Setiabudi`
* `Kabupaten Sragen 57262`
* `K.P. 124`

### 2.8 Contoh Tidak Valid

* `jalan tol ke bandara` → hanya street generik
* `Kelurahan, Kecamatan` → label tanpa nama
* `RT/RW` → tanpa angka
* `PC-12800 RAM 8GB` → angka 5 digit, tapi bukan ZIP (dan tanpa admin area)


## 3. Integrasi ke Pipeline

### Urutan Eksekusi

1. **Pre-clean**: hapus HTML, normalisasi whitespace, ganti koma/titik jadi spasi.
2. **Window builder**: gunakan **Trigger** untuk ekstrak window teks sekitar alamat.
3. **Validator (token proximity)**: hitung komponen regex, cek aturan validitas.
4. **Masking**: ganti **seluruh span window** yang valid dengan `[ADDR]`.

### Preset

* `strict` → gap=4
* `balanced` → gap=6
* `recall` → gap=8

## 4. Contoh End-to-End

### Contoh A — Alamat Valid

**Input**

```
Produk bagus, alamat toko di Jl. Teuku Umar No 76 Denpasar 80113, bisa cek map.
```

**Langkah**

* Trigger → `Jl.` → ambil window sekitar
* Komponen → `Street`, `House (No 76)`, `ZIP (80113)`
* Validator → House + ZIP berdekatan (≤ N token) → **valid**
* Masking → ganti span alamat jadi `[ADDR]`

**Output**

```
Produk bagus, alamat toko di [ADDR], bisa cek map.
```

