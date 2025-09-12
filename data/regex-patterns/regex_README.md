# `regex-patterns` - Quick Guide

This folder contains JSON regex dictionaries used by **Leksara** to detect, normalize, and mask Indonesian e-commerce text.

> * **`pii_rules.json` / `pii_patterns.json`**: regex for Personally Identifiable Information (PII) such as phone numbers, emails, IDs, and addresses.
> * **`rating_rules.json` / `rating_patterns.json`**: regex for star ratings and rating mentions in reviews.
> * **`regex_README.md`**: overview of this folder.
> * **`docs`**: detailed docs for specific topics (`rating_guide.md`, `regex_address.md`).

## File-by-file

### 1) `pii_rules.json`

**Purpose**

* Define PII regex for pipeline execution: phone, email, Indonesian IDs (NIK), and address components.
* Each rule includes `type`, `priority`, and optional `replace` for masking.

**Schema**

```json
{
  "schema_version": "1.0",
  "defaults": { "flags": ["IGNORECASE", "MULTILINE"] },
  "rules": [
    { "id": "...", "pattern": "...", "type": "extract/mask/trigger", "priority": 100 }
  ],
  "blacklist": [ { "id": "...", "pattern": "...", "reason": "..." } ]
}
```
**Fields**

* **id**: unique rule name (e.g., `pii_phone`).
* **pattern**: regex pattern string.
* **type**: how the rule is applied (`extract`, `mask`, or `trigger`).
* **replace**: optional replacement token (e.g., `[PHONE]`).
* **priority**: execution order (higher runs first).
* **desc**: short description of the rule.
* **example**: sample text that matches.
* **value\_group** *(optional)*: regex group index used to extract value.
* **normalize** *(optional)*: post-processing step (e.g., `phone`, `email`).

**Example entries (excerpt)**

```json
    {
      "id": "pii_phone",
      "pattern": "\\b(?:\\+62|62|0)(?:\\d{2,3}[- ]?){2,4}\\d{3,4}\\b",
      "example": "+6281234567890",
      "type": "extract",
      "value_group": 0,
      "normalize": "phone",
      "priority": 100,
      "desc": "Nomor HP Indonesia umum"
    }
```

### 2) `pii_patterns.json`

**Purpose**

* Human-readable reference of PII regex.
* Easier to browse patterns with `id`, `pattern`, `desc`, and `example`.

**Schema**

```json
[
  { "id": "<rule_id>", "pattern": "<regex>", "desc": "<short_note>", "example": "<example_text>" }
]
```
**Fields**

* **id**: identifier linking to the same rule in `pii_rules.json`.
* **pattern**: regex string only (no execution config).
* **desc**: human-readable explanation of what the regex matches.
* **example**: sample string that should match the regex.
**Example entries (excerpt)**

```json
[
  {
    "id": "pii_phone",
    "pattern": "\\b(?:\\+62|62|0)(?:\\d{2,3}[- ]?){2,4}\\d{3,4}\\b",
    "desc": "Nomor HP Indonesia umum",
    "example": "+6281234567890"
  }
]
```

### 3) `rating_rules.json`

**Purpose**

* Regex rules for detecting rating mentions in reviews (e.g., `5/5`, `kasih 4 bintang`, `⭐⭐⭐⭐`).
* Used to normalize ratings into `[RATING]`.

**Schema**

```json
{
  "rules": [
    { "id": "rating_star", "pattern": "...", "replace": "[RATING]" }
  ]
}
```

**Example entries (excerpt)**

```json
    {
      "id": "frac_out_of_5",
      "pattern": "\\b([1-5](?:[.,]5)?)\\s*/\\s*5\\b",
      "example": "4.5/5 mantap",
      "type": "extract",
      "value_group": 1,
      "postprocess": {
        "replace": {
          ",": "."
        }
      },
      "clamp": [
        1.0,
        5.0
      ],
      "priority": 100
    }
```

### 4) `rating_patterns.json`

**Purpose**

* Documentation-only version of rating regex, similar to `pii_patterns.json`.
* Includes examples for easier review.

**Schema**

```json
[
  { "id": "rating_star", "pattern": "...", "example": "⭐⭐⭐" }
]
```

**Example entries (excerpt)**

```json
[
  {
    "id": "frac_out_of_5",
    "pattern": "\\b([1-5](?:[.,]5)?)\\s*/\\s*5\\b",
    "example": "4.5/5 mantap"
  }
]
```

## Special Notes

* **Address Regex**: recommended to apply validation (≥2 components, proximity check). See [`regex_address.md`](./docs/regex_address.md).
* **Ratings Regex**: more detail in [`rating_guide.md`](./docs/rating_guide.md).
