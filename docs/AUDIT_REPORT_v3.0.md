# PIID — Benchmark Dataset Audit Report
## Comprehensive Integrity & Validation Report (v3.0 — Public Release)

**Audit Version:** 3.0 (Authoritative Public Release)  
**Release Directory:** `PIID-PUBLIC-RELEASE/`  
**Dataset Path:** `data/`  
**Audit Scope:** Full binary inspection, SHA-256 checksum verification, row-level schema validation, modality arithmetic, cross-split leakage testing, and path integrity.

---

## 1. Executive Summary

| Verification Metric | Target / Specification | Actual Audited Value | Status |
|---|---|---|---|
| **Total Benchmark Records** | 289,874 | **289,874** | ✅ **PERFECT MATCH** |
| **Benign Class Count** | 144,937 (50.00%) | **144,937** (50.00%) | ✅ **PERFECT MATCH** |
| **Malicious Class Count** | 144,937 (50.00%) | **144,937** (50.00%) | ✅ **PERFECT MATCH** |
| **Attack Families** | 15 (+ "none") | **15 (+ "none")** | ✅ **PERFECT MATCH** |
| **Unique Attack Payloads** | 58 (+ "none") | **58 (+ "none")** | ✅ **PERFECT MATCH** |
| **Train Partition Count** | 231,900 (80.00%) | **231,900** (80.00%) | ✅ **PERFECT MATCH** |
| **Validation Partition Count** | 28,986 (10.00%) | **28,986** (10.00%) | ✅ **PERFECT MATCH** |
| **Test Partition Count** | 28,988 (10.00%) | **28,988** (10.00%) | ✅ **PERFECT MATCH** |
| **Parquet Sum (Train + Val + Test)** | 289,874 | **289,874** | ✅ **EXACT MATCH** |
| **Schema Uniformity** | 9 uniform columns | **9 uniform columns across all 3 splits** | ✅ **100% UNIFORM** |
| **Sample ID Formatting** | Uniform canonical ID | `benign_NNNNNNNN` / `malicious_NNNNNNNN` | ✅ **100% UNIFORM** |
| **Class Label Encoding** | Canonical string | strictly `'benign'` or `'malicious'` | ✅ **100% UNIFORM** |
| **Text-Only Records** | 288,004 (99.355%) | **288,004** (`image_path` is null) | ✅ **VERIFIED** |
| **Multimodal Records** | 1,870 (0.645%) | **1,870** (`image_path` points to PNG) | ✅ **VERIFIED** |
| **Physical Images on Disk** | 1,870 | **1,870 PNG files in `data/images/`** | ✅ **100% RESOLVABLE** |
| **Broken Image Paths** | 0 | **0 broken paths** (100% exist on disk) | ✅ **ZERO DEFECTS** |
| **Cross-Split ID Leakage** | 0 | **0 duplicate sample IDs across splits** | ✅ **ZERO LEAKAGE** |
| **Source-Pair Group Leakage** | 0 | **0 group pairs split across sets** | ✅ **ZERO LEAKAGE** |

---

## 2. Partition & Split Verification

All data splits were partitioned using scikit-learn `GroupShuffleSplit` (seed=42) keyed on the numeric ID pair (`benign_NNNNNNNN` and `malicious_NNNNNNNN` share group key `NNNNNNNN`). This strictly guarantees that no benign source document and its paired adversarial injection are distributed across different evaluation sets.

### Split Summary

```
BENCHMARK TOTAL: 289,874 records
├── data/train.parquet:       231,900 records (80.00%)  [115,950 benign, 115,950 malicious]
│                             └── 1,504 multimodal | 230,396 text-only
├── data/validation.parquet:   28,986 records (10.00%)  [ 14,493 benign,  14,493 malicious]
│                             └──   184 multimodal |  28,802 text-only
└── data/test.parquet:         28,988 records (10.00%)  [ 14,494 benign,  14,494 malicious]
                              └──   182 multimodal |  28,806 text-only
```

### Mutual Exclusivity and Leakage Checks
- **Train ∩ Validation Overlap:** **0 rows**
- **Train ∩ Test Overlap:** **0 rows**
- **Validation ∩ Test Overlap:** **0 rows**
- **Group Key Cross-Partition Overlap:** **0 groups (0 pairs)**
- **Total Unique `sample_id`s in Parquets:** **289,874**

---

## 3. Schema & Column Specifications

All three split Parquet files share the exact same 9 columns with identical naming, order, and data types:

| # | Column Name | Physical Type | Logical Nulls | Description |
|---|---|---|---|---|
| 1 | `sample_id` | `string` | 0 | Canonical record identifier (`benign_NNNNNNNN` / `malicious_NNNNNNNN`) |
| 2 | `text` | `string` | 0 | Raw textual content of document or injected prompt |
| 3 | `ocr_text` | `string` | 0 | OCR-extracted text (populated for image samples; duplicated/empty as applicable) |
| 4 | `image_path` | `string / null` | 288,004 | Relative path (`data/images/<file>.png`) for multimodal samples; null for text-only |
| 5 | `label` | `string` | 0 | Ground-truth class: strictly `'benign'` or `'malicious'` |
| 6 | `attack_family` | `string` | 0 | Category (15 attack families for malicious, `'none'` for benign) |
| 7 | `payload_id` | `string` | 0 | Payload identifier (`PAY-*` for malicious, `'none'` for benign) |
| 8 | `source_module` | `string` | 0 | Source module: `MASTER_BENIGN_DATASET` or `MASTER_MALICIOUS_DATASET` |
| 9 | `split` | `string` | 0 | Split identifier: `'train'`, `'validation'`, or `'test'` |

---

## 4. Image Inventory & Path Sanitization

During initial pipeline generation in Google Colab, Module C generated 3,687 composite image artifacts. 
- **1,870 images** were retained in storage and are included in `data/images/`.
- **1,817 historical references** were not retained in local storage.

### Release Sanitization Performed:
1. **Path Normalization:** All 1,870 valid image references were normalized from legacy paths to standard relative paths of the form `data/images/<filename>.png`.
2. **Missing Reference Nullification:** The 1,817 historical image references pointing to unreleased files were sanitized to `null` in the parquet `image_path` field. All corresponding text and OCR text content were fully preserved, converting these records into valid text-only samples.
3. **Integrity Audit:** 
   - Non-null `image_path` values in parquets: exactly 1,870.
   - Files physically present on disk in `data/images/`: exactly 1,870.
   - Broken image paths: **0**.
   - Absolute or environment-dependent paths: **0**.

---

## 5. Attack Taxonomy Distribution

All 144,937 malicious records strictly map to the 15 documented attack families:

| Attack Family | Sample Count |
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
| **Total Malicious** | **144,937** |

---

## 6. Authoritative SHA-256 Checksums

The release Parquet split files have been verified with the following cryptographic SHA-256 hashes:

| File | SHA-256 Checksum |
|---|---|
| `data/train.parquet` | `da60b014bb648d29dcbdb26f1f682496d020915c9bc0031b79bd0067aa1755d0` |
| `data/validation.parquet` | `b2a0c59892244475fb2d019332faf8ae50c53deb1aad850a329988453f186f01` |
| `data/test.parquet` | `685d1461cfee35030a167bd58b046c80214f676af7d1522ae1d3eeceaeb5e243` |

All 1,870 PNG images in `data/images/` and the 3 Parquet files are catalogued in `CHECKSUMS.sha256` (1,873 verified entries).

---

## 7. Audit Sign-Off

The public release directory `PIID-PUBLIC-RELEASE/` satisfies all 13 dataset readiness criteria:

1. `required_columns_metadata`: ✅ PASS (All 9 columns present)
2. `no_missing_values_core_fields`: ✅ PASS (Zero nulls in core identifier and text keys)
3. `no_duplicate_sample_ids`: ✅ PASS (289,874 unique sample IDs)
4. `no_duplicate_rows_unified`: ✅ PASS (Zero duplicate rows)
5. `label_consistency`: ✅ PASS (`'benign'` and `'malicious'` only)
6. `payload_consistency`: ✅ PASS (58 payloads + `'none'`)
7. `attack_family_consistency`: ✅ PASS (15 families + `'none'`)
8. `metadata_consistency`: ✅ PASS (Exact match with `dataset_manifest.json`)
9. `split_consistency`: ✅ PASS (Train=231,900, Val=28,986, Test=28,988)
10. `csv_vs_parquet_consistency`: ✅ PASS (All parquet records match sample catalog)
11. `modality_consistency`: ✅ PASS (288,004 text-only, 1,870 multimodal)
12. `cross_split_leakage`: ✅ PASS (Zero sample ID overlap, zero group leakage)
13. `path_integrity`: ✅ PASS (Zero broken image paths, all relative)

**Audit Status:** **APPROVED FOR DEPOSIT (Mendeley Data)**
