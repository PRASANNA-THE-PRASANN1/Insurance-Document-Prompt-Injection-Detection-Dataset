# FINAL RELEASE VALIDATION — PIID v1.0.0
**Validation Date:** 2026-10-01  
**Release Directory:** `PIID-PUBLIC-RELEASE/`  
**Target Repository:** Mendeley Data  

---

## 1. Dataset Integrity Verification

### Row Counts (Authoritative)

| Split | Expected | Actual | Status |
|---|---|---|---|
| `data/train.parquet` | 231,900 | **231,900** | ✅ PASS |
| `data/validation.parquet` | 28,986 | **28,986** | ✅ PASS |
| `data/test.parquet` | 28,988 | **28,988** | ✅ PASS |
| **Total** | **289,874** | **289,874** | ✅ PASS |

### Class Balance

| Split | Benign | Malicious | Status |
|---|---|---|---|
| train | 115,950 | 115,950 | ✅ PASS |
| validation | 14,493 | 14,493 | ✅ PASS |
| test | 14,494 | 14,494 | ✅ PASS |
| **Total** | **144,937** | **144,937** | ✅ PASS |

### Schema (All 3 Splits Identical)

| Column | Data Type | Null Count | Description |
|---|---|---|---|
| `sample_id` | str | 0 | Canonical identifier (`benign_NNNNNNNN` / `malicious_NNNNNNNN`) |
| `text` | str | 0 | Document text / prompt content |
| `ocr_text` | str | 0 | OCR-extracted text |
| `image_path` | str | 288,004 | Relative path (`data/images/<file>.png`) for 1,870 multimodal records; null for 288,004 text-only records |
| `label` | str | 0 | Class label: `'benign'` or `'malicious'` |
| `attack_family` | str | 0 | 15 families for malicious, `'none'` for benign |
| `payload_id` | str | 0 | 58 payloads for malicious, `'none'` for benign |
| `source_module` | str | 0 | `MASTER_BENIGN_DATASET` or `MASTER_MALICIOUS_DATASET` |
| `split` | str | 0 | `'train'`, `'validation'`, or `'test'` |

### Taxonomy

| Metric | Expected | Actual | Status |
|---|---|---|---|
| Attack families | 15 + "none" | **15 + "none"** | ✅ PASS |
| Payloads | 58 + "none" | **58 + "none"** | ✅ PASS |
| Labels | benign, malicious | **benign, malicious** | ✅ PASS |

### Integrity Checks

| Check | Result | Status |
|---|---|---|
| Duplicate sample_ids | 0 | ✅ PASS |
| Train ∩ Validation overlap | 0 | ✅ PASS |
| Train ∩ Test overlap | 0 | ✅ PASS |
| Validation ∩ Test overlap | 0 | ✅ PASS |
| Source-pair group leakage | 0 | ✅ PASS |
| Broken image paths | 0 | ✅ PASS |
| Non-standard / absolute paths | 0 | ✅ PASS |

---

## 2. Image Inventory & Modality

| Metric | Value | Status |
|---|---|---|
| Physical PNG files in `data/images/` | **1,870** | ✅ VERIFIED |
| Parquet rows with non-null `image_path` | **1,870** | ✅ VERIFIED |
| Parquet rows with null `image_path` (text-only) | **288,004** | ✅ VERIFIED |
| Historical unreleased image references sanitized to null | **1,817** | ✅ SANITIZED |
| Checksum entries in `CHECKSUMS.sha256` | **1,873** (3 parquets + 1,870 images) | ✅ VERIFIED |

---

## 3. Cryptographic Checksum Verification

| File | SHA-256 Checksum | Status |
|---|---|---|
| `data/train.parquet` | `da60b014bb648d29dcbdb26f1f682496d020915c9bc0031b79bd0067aa1755d0` | ✅ VERIFIED |
| `data/validation.parquet` | `b2a0c59892244475fb2d019332faf8ae50c53deb1aad850a329988453f186f01` | ✅ VERIFIED |
| `data/test.parquet` | `685d1461cfee35030a167bd58b046c80214f676af7d1522ae1d3eeceaeb5e243` | ✅ VERIFIED |
| All 1,870 images | Catalogued in `CHECKSUMS.sha256` | ✅ VERIFIED |

**Note on Parquet Updates:** Parquet files were updated during the final release pass to relativize the 1,870 image paths to `data/images/<filename>.png` and nullify the 1,817 historical Colab image references where image files were not retained. This eliminated 100% of broken and environment-dependent paths while preserving all underlying document text and OCR text.

---

## 4. Path Sanitization

| Scan Target | Hits Before | Hits After | Status |
|---|---|---|---|
| Local user machine paths in docs/metadata | Detected | **0** | ✅ CLEANED |
| Private URL / filesystem links | Detected | **0** | ✅ CLEANED |
| Colab paths in `image_manifest.csv` | 3,687 entries | **0** | ✅ CLEANED |
| Colab paths in Parquet `image_path` | 3,687 entries | **0** (1,870 relative, 1,817 null) | ✅ CLEANED |
| Broken image paths in Parquets | 1,817 | **0** | ✅ CLEANED |

---

## 5. Secret & Credential Scan

| Scan Target | Result |
|---|---|
| API keys / Access tokens | None found |
| Passwords / Private secrets | None found |
| Private URLs / Cloud bucket credentials | None found |
| Personal email addresses / usernames | None found |

---

## 6. Documentation & Release File Alignment

| File | Verification Item | Status |
|---|---|---|
| `README.md` | Authoritative counts, quick start, relative image loading | ✅ VERIFIED |
| `DATASET_CARD.md` (root + docs/) | HF YAML metadata, accurate modality counts, clean schema | ✅ VERIFIED |
| `docs/AUDIT_REPORT_v3.0.md` | Cleaned audit report matching verified data | ✅ VERIFIED |
| `docs/DATA_PROVENANCE.md` | Upstream corpora details, license obligations, image notes | ✅ VERIFIED |
| `docs/PIPELINE_OVERVIEW.md` | Five-module pipeline description, output directory structure | ✅ VERIFIED |
| `docs/LICENSE_NOTES.md` | Third-party source terms, CC BY 4.0 composite terms | ✅ VERIFIED |
| `metadata/dataset_manifest.json` | Fully populated JSON manifest matching all splits & hashes | ✅ VERIFIED |
| `metadata/image_manifest.csv` | 1,870 released image entries matching `data/images/` | ✅ VERIFIED |
| `metadata/validation_report.json` | All 20 validation checks pass | ✅ VERIFIED |
| `CHECKSUMS.sha256` | 1,873 entries with exact matching hashes | ✅ VERIFIED |
| `CITATION.cff` | Correct counts and citation metadata | ✅ VERIFIED |

---

## 7. Mendeley Deposit Readiness

| Criterion | Status |
|---|---|
| All documentation consistent on final numbers | ✅ YES |
| No personal/machine paths in public files | ✅ YES |
| No secrets or credentials | ✅ YES |
| No broken image paths in parquets | ✅ YES |
| Checksums verified and aligned | ✅ YES |
| Licensing caveats clearly stated | ✅ YES |
| Pipeline consistently described as 5-module | ✅ YES |
| **READY FOR DEPOSIT** | ✅ **YES** |

---

*Generated: 2026-10-01 | Validation method: Automated parquet inspection + cryptographic checksum verification*
