# Insurance Document Prompt-Injection Detection Dataset

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![DOI](https://img.shields.io/badge/DOI-10.17632%2F5b6xz84n79.1-blue.svg)](https://data.mendeley.com/datasets/5b6xz84n79/1)
**Version:** 1.0.0 | **Pipeline:** Five-Module (A → B → C → D → E)  
**Mendeley Data:** [10.17632/5b6xz84n79.1](https://data.mendeley.com/datasets/5b6xz84n79/1) | **GitHub:** [PRASANNA-THE-PRASANN1/Insurance-Document-Prompt-Injection-Detection-Dataset](https://github.com/PRASANNA-THE-PRASANN1/Insurance-Document-Prompt-Injection-Detection-Dataset)

---

## Overview

**PIID** is a large-scale, reproducible benchmark dataset
for **prompt-injection detection** in insurance AI systems.

| Property | Value |
|---|---|
| Total samples | **289,874** |
| Benign / Malicious | **144,937 / 144,937** (balanced 50/50) |
| Format / Subsets | Predominantly text-based (288,004 records) + retained document images (1,870 records) |
| Attack families | **15** |
| Unique payloads | **58** |
| Train / Val / Test | **231,900 / 28,986 / 28,988** |
| Parquet schema | 9 columns (unified across all splits) |
| Random seed | 42 |

The release is predominantly text-based, with 1,870 records retaining associated document images.

---

## Quick Start

### Load the Dataset (Python)

```python
import pandas as pd

# Load splits
df_train = pd.read_parquet("data/train.parquet")
df_val   = pd.read_parquet("data/validation.parquet")
df_test  = pd.read_parquet("data/test.parquet")

print(df_train.shape)  # (231900, 9)
print(df_train["label"].value_counts())
# malicious    115950
# benign       115950
```

### Label Encoding (for ML)

```python
label_map = {"benign": 0, "malicious": 1}
df_train["label_int"] = df_train["label"].map(label_map)
```

### Load with Images

```python
from PIL import Image
import os

def load_sample(row, release_root="."):
    text = row["text"]
    if pd.notna(row["image_path"]):
        img = Image.open(os.path.join(release_root, row["image_path"]))
    else:
        img = None
    return text, img
```

---

## Directory Structure

```
PIID-PUBLIC-RELEASE/
│
├── data/
│   ├── train.parquet          (231,900 rows)
│   ├── validation.parquet      (28,986 rows)
│   ├── test.parquet            (28,988 rows)
│   └── images/                (1,870 PNG files)
│
├── metadata/
│   ├── dataset_manifest.json  (complete manifest with SHA-256 checksums)
│   ├── sample_index.csv       (lightweight 6-col index, all 289,874 rows)
│   ├── attack_taxonomy.json   (15 families × 58 payloads structured taxonomy)
│   ├── image_manifest.csv     (1,870 released image path mapping)
│   └── validation_report.json (automated validation results)
│
├── code/
│   ├── 01_Module_A_FINAL.ipynb
│   ├── 02_Module_B_FINAL.ipynb
│   ├── 03_Module_C_FINAL.ipynb
│   ├── 04_Module_D_FINAL.ipynb
│   ├── 05_Module_E_FINAL.ipynb
│   ├── module_f_step0_fix_splits.py  (split correction utility)
│   └── requirements.txt
│
├── docs/
│   ├── DATA_PROVENANCE.md
│   ├── DATASET_CARD.md
│   ├── PIPELINE_OVERVIEW.md
│   ├── AUDIT_REPORT_v3.0.md
│   └── LICENSE_NOTES.md
│
├── CHECKSUMS.sha256
├── DATASET_CARD.md            (root copy for discoverability)
├── FINAL_RELEASE_VALIDATION.md
├── LICENSE
└── README.md                  (this file)
```

---

## Data Schema

All three splits share the **same 9-column schema**:

| Column | Type | Description |
|---|---|---|
| `sample_id` | string | `benign_NNNNNNNN` or `malicious_NNNNNNNN` |
| `text` | string | Document text (possibly injection-containing) |
| `ocr_text` | string | OCR-extracted text (for image samples) |
| `image_path` | string or null | Relative path to PNG in `data/images/`; null for text-only samples and for samples whose images are not included in this release |
| `label` | string | `"benign"` or `"malicious"` |
| `attack_family` | string | Attack category (15 families or `"none"`) |
| `payload_id` | string | `PAY-*` identifier or `"none"` |
| `source_module` | string | `MASTER_BENIGN_DATASET` or `MASTER_MALICIOUS_DATASET` |
| `split` | string | `"train"`, `"validation"`, or `"test"` |

---

## Attack Taxonomy

15 adversarial attack families targeting insurance AI systems:

| Attack Family | Malicious Samples |
|---|---|
| Context Poisoning | 9,663 |
| Instruction Override | 9,663 |
| Footer Injection | 9,663 |
| Policy Bypass | 9,663 |
| White Text Injection | 9,663 |
| Metadata Injection | 9,663 |
| Insurance-specific Attacks | 9,663 |
| Hidden Multimodal | 9,662 |
| Header Injection | 9,662 |
| Data Exfiltration | 9,662 |
| Decision Manipulation | 9,662 |
| OCR-specific Attacks | 9,662 |
| Invisible Character Injection | 9,662 |
| Tiny Font Injection | 9,662 |
| Watermark Injection | 9,662 |

See `metadata/attack_taxonomy.json` for complete payload-level breakdown.

---

## Reproducibility

The dataset was constructed using a five-module pipeline:

```
Module A → Module B → Module C → Module D → Module E
(Config)   (Text)     (Images)   (Validate) (Benchmark)
```

- **Random seed:** 42
- **Split method:** GroupShuffleSplit (scikit-learn)
- **Group key:** numeric suffix of `sample_id` (pairs `benign_NNNNNNNN` with `malicious_NNNNNNNN` in the same split)
- **Determinism:** Reproducible under the documented configuration and random seed, with potential environment-dependent variation in OCR and image rendering
- **Full pipeline:** See `docs/PIPELINE_OVERVIEW.md` and notebooks in `code/`

---

## Integrity Verification

Verify all data files using:

```bash
# Linux/macOS
sha256sum -c CHECKSUMS.sha256

# Windows (PowerShell)
Get-Content CHECKSUMS.sha256 | ForEach-Object {
    $hash, $file = $_ -split '  ', 2
    $actual = (Get-FileHash $file -Algorithm SHA256).Hash.ToLower()
    if ($actual -eq $hash) { Write-Host "OK: $file" }
    else { Write-Host "FAIL: $file" }
}
```

**Authoritative SHA-256 checksums:**
```
train.parquet:      da60b014bb648d29dcbdb26f1f682496d020915c9bc0031b79bd0067aa1755d0
validation.parquet: b2a0c59892244475fb2d019332faf8ae50c53deb1aad850a329988453f186f01
test.parquet:       685d1461cfee35030a167bd58b046c80214f676af7d1522ae1d3eeceaeb5e243
```

---

## License

**Composite licensing:**
- **Project-created components** (synthetic text, generated images, metadata, code, pipeline notebooks): [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- **Third-party-derived material** incorporated into generated images: subject to the terms of the originating sources. See `docs/LICENSE_NOTES.md` for details.
- Users are responsible for reviewing applicable third-party terms before redistribution or commercial use.

---

## Citation

If you use this dataset, please cite:

```bibtex
@dataset{piid_2026,
  title        = {Insurance Document Prompt-Injection Detection Dataset},
  author       = {Saxena, Prasanna},
  year         = {2026},
  version      = {1.0.0},
  publisher    = {Mendeley Data},
  doi          = {10.17632/5b6xz84n79.1},
  url          = {https://data.mendeley.com/datasets/5b6xz84n79/1},
  howpublished = {\url{https://data.mendeley.com/datasets/5b6xz84n79/1}},
  note         = {Five-module synthetic pipeline (A-B-C-D-E). 289,874 samples, 15 attack families, 58 payloads. Seed=42.}
}
```

---

## Audit & Integrity

Full dataset integrity has been verified. See `docs/AUDIT_REPORT_v3.0.md` and `FINAL_RELEASE_VALIDATION.md`.

Key validation results:
- Zero cross-split sample-ID leakage (train ∩ val ∩ test = ∅)
- Zero source-pair group leakage across splits
- Full schema uniformity across all 3 splits (9 columns)
- 289,874 unique sample_ids confirmed
- All 1,870 image paths resolve to physical PNG files
- Zero broken or absolute paths in released parquets
