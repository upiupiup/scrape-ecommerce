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

### 5) `emoji_dictionary.json`

**Purpose**
- Map common used emojis in reviews or product names to a *canonical* Indonesian form for consistent matching

**Schema**
```json
{
  "<emoji>": "<mapping_bahasa_indonesia>"
}
```

**Example entries (except)**
```json
{
    "👍": "bagus",
    "🙏": "terima kasih",
    "🥰": "suka banget",
    "😍": "suka banget",
    "😁": "senang / gembira",
    "🫶": "cinta / kasih sayang / support",
    "😭": "sedih / terharu",
    "❤": "cinta / suka",
    "😊": "senyum / bahagia"
}
```

Oke, biar jelas, aku tambahin bagian **Field Descriptions** di bawah schema `dictionary_rules.json`. Jadi orang tim langsung tahu tiap field dipakai buat apa 👇

---

### 6) `dictionary_rules.json`

**Purpose**

* Handle **conflicts** in dictionary entries where a token has multiple possible meanings.
* Rules are defined per section (e.g., `acronym`, `slangs`) so that context decides which meaning is correct.
* Prevents mis-normalization (e.g., `m` could mean *meter* or *medium*).

**Schema**

```json
{
  "schema_version": "1.0",
  "sections": {
    "<dictionary_section>": {
      "conflict_rules": [
        {
          "id": "<unique_rule_id>",
          "token": "<ambiguous_token>",
          "candidates": ["<meaning1>", "<meaning2>"],
          "rules": [
            {
              "context_pattern": "<regex_pattern>",
              "preferred": "<chosen_meaning>",
              "desc": "<human_readable_explanation>"
            }
          ]
        }
      ],
      "priority_order": ["whitelist", "acronym", "slang"]
    }
  }
}
```

**Field Descriptions**

* **`schema_version`** → versioning control, in case schema changes in the future.
* **`sections`** → grouping by dictionary type (e.g., `"acronym"`, `"slangs"`).
* **`conflict_rules`** → list of rules for tokens that have more than one meaning.

  * **`id`** → unique identifier for the rule (string).
  * **`token`** → the ambiguous dictionary key (e.g., `"m"`, `"l"`, `"s"`).
  * **`candidates`** → all possible meanings for that token.
  * **`rules`** → how to resolve the conflict based on context.

    * **`context_pattern`** → regex used to detect usage context.
    * **`preferred`** → which candidate to pick if `context_pattern` matches.
    * **`desc`** → explanation in human-readable form.
* **`priority_order`** → global priority if conflicts remain across different dictionaries (example: always trust whitelist > acronym > slang).

**Example entries (excerpt)**

```json
{
  "schema_version": "1.0",
  "sections": {
    "acronym": {
      "conflict_rules": [
        {
          "id": "m_unit_vs_size",
          "token": "m",
          "candidates": ["meter", "medium"],
          "rules": [
            {
              "context_pattern": "(\\d+)\\s*m\\b",
              "preferred": "meter",
              "desc": "If 'm' follows a number, interpret as unit meter."
            },
            {
              "context_pattern": "\\b(size|ukuran|baju)\\s*m\\b",
              "preferred": "medium",
              "desc": "If 'm' is preceded by size indicators, interpret as size Medium."
            }
          ]
        }
      ],
      "priority_order": ["whitelist", "acronym", "slang"]
    }
  }
}
```
