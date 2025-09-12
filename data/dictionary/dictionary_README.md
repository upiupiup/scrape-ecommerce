# `dictionary` - Quick Guide

This folder contains JSON dictionaries used by **Leksara** to standardize and clean Indonesian e‑commerce text.

> - **`acronym_dict.json`**: standardize technical abbreviations/units/short forms.  
> - **`contractions_dict.json`**: expand Indonesian chat contractions (e.g., `gk` → `tidak`).  
> - **`slangs_dict.json`**: normalize slang/informal words (e.g., `kerenz` → `bagus`).  
> - **`whitelist.json`**: tokens that must be preserved during cleaning (e.g., `[PHONE]`, rating tags).

## File-by-file

### 1) `acronym_dict.json`
**Purpose**
- Map common abbreviations, measurement units, and short technical terms to a *canonical* Indonesian form for consistent matching/search.

**Schema**
```json
{
  "<short_or_unit>": "<canonical_form>"
}
```
- **Keys**: lowercased short forms or units (e.g., `"ghz"`, `"dpi"`, `"5mm"`).
- **Values**: Indonesian canonical text (e.g., `"gigahertz"`, `"titik per inci"`, `"5 milimeter"`).

**Example entries (excerpt)**
```json
{
  "ghz": "gigahertz",
  "dpi": "titik per inci",
  "5mm": "5 milimeter",
  "hdr": "High Dynamic Range",
  "amoled": "Active Matrix Organic Light Emitting Diode"
}
```

### 2) `contractions_dict.json`
**Purpose**
- Expand common Indonesian chat contractions to formal forms, improving downstream tokenization and matching.

**Schema**
```json
{
  "<contraction>": "<expanded_form>"
}
```
- **Keys**: common short forms (`"gk"`, `"yg"`, `"dr"`, `"sdh"`, `"dst"`).
- **Values**: formal Indonesian (`"tidak"`, `"yang"`, `"dari"`, `"sudah"`, `"dan seterusnya"`).

**Example entries (excerpt)**
```json
{
  "gk": "tidak",
  "gak": "tidak",
  "yg": "yang",
  "dr": "dari",
  "sdh": "sudah",
  "blm": "belum",
  "klo": "kalau"
}
```

### 3) `slangs_dict.json`
**Purpose**
- Normalize slang/informal variants to a neutral, canonical Indonesian form to stabilize vocab for models and search.

**Schema**
```json
{
  "<slang_or_variant>": "<canonical_form>"
}
```
- **Keys**: lowercased slang/noisy variants (e.g., `"alayyyyy"`, `"parahh"`, `"yaaa"`).
- **Values**: neutral canonical forms (e.g., `"norak"`, `"parah"`, `"iya"`).

**Example entries (excerpt)**
```json
{
  "keren": "bagus",
  "kepo": "penasaran",
  "sip": "oke",
  "parahh": "parah",
  "gas": "lanjut",
  "kerenz": "bagus"
}
```


### 4) `whitelist.json`
**Purpose**
- Protect special tokens from being altered by cleaning steps (e.g., placeholders or tags used by upstream masking).

**Schema**
```json
{
  "SYSTEM_PLACEHOLDERS": ["<TOKEN_1>", "<TOKEN_2>", "..."],
  "RATING_TOKENS": ["<RATING_1>", "...", "<RATING_5>"]
}
```

**Example entries (excerpt)**
```json
{
  "SYSTEM_PLACEHOLDERS": ["[PHONE]", "[EMAIL]", "[ADDR]", "[ID]", "[CARD]", "[ORDER_CODE]"],
  "RATING_TOKENS": ["[RATING_1]", "[RATING_2]", "[RATING_3]", "[RATING_4]", "[RATING_5]"]
}
```


**Last updated:** v1
