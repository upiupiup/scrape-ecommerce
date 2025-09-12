# Regex Notes - Indonesian Address

## Purpose

Detect real Indonesian addresses in noisy e-commerce text (descriptions, reviews) and mask them as `[ADDR]`.

## Tringger and Component Address Patterns

* **Pattern (Trigger)**
  Acts as an **alarm**.
  It’s a lightweight regex that looks for common address keywords (like `Jl.`, `RT`, `Kelurahan`).
  → Used to decide *“this part of text might contain an address”* and to extract a **window** around it.

* **Component**
  Acts as the **details**.
  Each regex detects one type of administrative unit inside the window (house number, RT/RW, kelurahan, etc.).
  → Used to decide *“is this really an address, and is it complete enough?”*

📌 **Workflow:**
Trigger finds → Components are checked → Validation rules decide **valid/invalid**.

## Regex Components

* **Street**:
  `Jl./Jalan/Gg./Gang + name`
  *(not counted as admin component)*

* **House Number**:
  `No/Nomor/Nmr/# + digits/letters`

* **RT/RW**:
  `RT xx RW yy` (both required as numbers)

* **Kelurahan**:
  `Kel/ Kelurahan + name`

* **Kecamatan**:
  `Kec/ Kecamatan + name`

* **Kabupaten/Kota**:
  `Kab/Kabupaten/Kota + name`

* **Province**:
  `Prov/Provinsi + name`

* **ZIP (Kode Pos)**:
  `5-digit number`
  *(must be paired with admin components, never stand-alone)*

## Trigger Pattern

```regex
\b(Jl\.?|Jalan|Gg\.?|Gang|RT|RW|Kel(?:urahan)?\.?|Kec(?:amatan)?\.?|Kab(?:upaten)?\.?|Kota|Prov(?:insi)?)\b
```

ZIP is **not included** in trigger because 5-digit numbers often appear as prices/SKUs.

## Validation Rules

An address is considered **valid** if:

1. At least **two administrative components** are present
   (house, RT/RW, kel, kec, kab/kota, prov, zip).

2. Components appear **close to each other** (≤ *N* tokens apart).

3. **ZIP must be paired** with at least one admin component (kel/kec/kab/prov).

4. **Reject empty labels**:

   * `Kelurahan` / `Kecamatan` without a name
   * `RT/RW` without numbers

## Token Proximity

* Default: `MAX_TOKEN_GAP = 6` (balanced)
* Stricter: `MAX_TOKEN_GAP = 4`
* Looser recall: `MAX_TOKEN_GAP = 8`

## Examples

✅ **Valid**

* `Jl. Teuku Umar No 76 Denpasar 80113`
* `Kelurahan Melati, Kecamatan Setiabudi`
* `Kabupaten Sragen 57262`

❌ **Invalid**

* `jalan tol ke bandara` (street only)
* `Kelurahan, Kecamatan` (labels with no names)
* `RT/RW` (no numbers)
* `PC-12800 RAM 8GB` (5-digit number but not ZIP)

## Pipeline Usage

1. **Pre-clean**: normalize whitespace, remove HTML, convert punctuation → spaces.
2. **Trigger (Pattern)**: detect candidate windows.
3. **Components**: run regex components inside windows.
4. **Validation**: apply rules (≥2 components, proximity, zip check).
5. **Masking**: replace entire window with `[ADDR]`.
