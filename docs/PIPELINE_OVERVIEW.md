# Pipeline Overview — PIID Five-Module Architecture
**Version:** 1.0.0 | **Project:** PIID  
**Pipeline Architecture:** Five-Module (A → B → C → D → E) + Release Verification Step  
**Target Repository:** Mendeley Data

---

## 1. Architecture Summary

The PIID benchmark dataset was produced through a sequential five-module architecture (Modules A through E). Each module fulfills a specific generation responsibility and produces frozen intermediate outputs consumed by subsequent stages.

Following pipeline execution, a dedicated release-preparation verification and split-correction step was executed to establish group-leakage-free partitions and package the authoritative public release.

```
RAW SOURCES (CORD, DocVQA, LIC, NAIC, RVL-CDIP)
         │
         ▼
  [ MODULE A ] ─── Analysis, Configuration, Payload Bank
         │               Output: payload_bank.json (58 payloads, 15 families)
         ▼
  [ MODULE B ] ─── One-to-One Text Injection
         │               Output: malicious_text.csv (144,937 injected records)
         ▼
  [ MODULE C ] ─── Multimodal Artifact Generation (Images, Renderings, Labels)
         │               Output: 3,687 composite image artifacts generated during development
         ▼
  [ MODULE D ] ─── Dataset Validation + Grouped Splits + Parquet Builder
         │               Output: Validated malicious dataset partition
         ▼
  [ MODULE E ] ─── Benchmark Dataset Builder
                         Input:  MASTER_BENIGN_DATASET + MASTER_MALICIOUS_DATASET
                         Output: Consolidated benchmark dataset
                                 │
                                 ▼
         [ RELEASE PREPARATION & VERIFICATION UTILITY ]
               (code/module_f_step0_fix_splits.py)
                         │
                         ▼
             AUTHORITATIVE PUBLIC RELEASE ARTIFACTS:
             ├── data/train.parquet       (231,900 rows)
             ├── data/validation.parquet   ( 28,986 rows)
             ├── data/test.parquet         ( 28,988 rows)
             ├── data/images/              (  1,870 PNG files)
             └── metadata/                 (manifests, taxonomy, index, report)
```

---

## 2. Module Specifications

### Module A — Analysis, Configuration, Payload Bank
- **Notebook:** `code/01_Module_A_FINAL.ipynb`
- **Scope & Responsibility:**
  - Analyzes raw upstream text and document corpora (CORD, DocVQA, LIC, NAIC, RVL-CDIP).
  - Formulates the 15 prompt-injection attack families.
  - Constructs the Insurance Payload Bank containing 58 unique prompt-injection payload templates targeting insurance operations (claims approval, coverage limits, policy exceptions).
  - Configures pipeline hyperparameters, random seed (42), and target split proportions (80/10/10).
- **Key Outputs:**
  - `payload_bank.json` (58 payloads across 15 families).
  - Configuration constants referenced by downstream modules.

**Attack Families Defined (15):**
Context Poisoning | Instruction Override | Footer Injection | Policy Bypass |
White Text Injection | Metadata Injection | Insurance-specific Attacks |
Hidden Multimodal | Header Injection | Data Exfiltration | Decision Manipulation |
OCR-specific Attacks | Invisible Character Injection | Tiny Font Injection |
Watermark Injection

---

### Module B — One-to-One Text Injection
- **Notebook:** `code/02_Module_B_FINAL.ipynb`
- **Scope & Responsibility:**
  - Ingests the frozen payload bank from Module A.
  - Executes a strict 1:1 injection design where each benign insurance document receives exactly one adversarial prompt injection.
  - Yields 144,937 malicious text samples, pairing each injected record with its source benign record identifier, payload template ID, and attack family.
- **Design Principle:** 1:1 pairing prevents representation imbalance and eliminates many-to-one mapping artifacts.
- **Key Outputs:**
  - `malicious_text.csv` (144,937 records).

---

### Module C — Multimodal Artifact Generation
- **Notebook:** `code/03_Module_C_FINAL.ipynb`
- **Scope & Responsibility:**
  - Consumes text injection records from Module B.
  - Renders document images in PNG format with adversarial instructions visually integrated into document backgrounds.
  - Synthesizes OCR-resilient injection variants (such as Tiny Font, Watermark, and Invisible Character patterns).
  - Generates label files and visual layout metadata.
- **Development Artifacts vs. Public Release Packaging:**
  - During development and pipeline execution, Module C generated **3,687 image artifacts**.
  - Exactly **1,870 images** were retained in storage and are physically included in the public release under `data/images/`.
  - The remaining **1,817 historical image references** were sanitized to `null` in the `image_path` field during public release packaging. They are preserved in the dataset as valid text-only records (with their full text and OCR text content intact). Zero broken or dangling image paths exist in the released dataset.

---

### Module D — Dataset Validation + Split Metadata Preparation + Parquet Builder
- **Notebook:** `code/04_Module_D_FINAL.ipynb`
- **Scope & Responsibility:**
  - Reads outputs from Modules A, B, and C as frozen, read-only artifacts.
  - Merges text and multimodal malicious metadata.
  - Validates schema conformity, verifies checksums, and eliminates duplicate entries.
  - **Prepares and validates preliminary split metadata** (train/validation/test assignments) using grouped partitioning logic to prevent payload template overlap across splits.
  - Compiles validated malicious dataset partitions into Parquet format.
  - **Note:** Module D's split assignments are preliminary. The final authoritative split partitions were established and reconciled by the release-preparation utility (`code/module_f_step0_fix_splits.py`), described in Section 3 below.

---

### Module E — Benchmark Dataset Builder
- **Notebook:** `code/05_Module_E_FINAL.ipynb`
- **Scope & Responsibility:**
  - Ingests standardized benign document records (`MASTER_BENIGN_DATASET`) alongside validated malicious records from Module D (`MASTER_MALICIOUS_DATASET`).
  - Standardizes the uniform 9-column schema across all instances:
    `sample_id`, `text`, `ocr_text`, `image_path`, `label`, `attack_family`, `payload_id`, `source_module`, `split`.
  - Merges instances into the complete 289,874-record benchmark.

---

## 3. Release Preparation & Verification Utility

- **Script:** `code/module_f_step0_fix_splits.py`
- **Role:** **Release-preparation, reconciliation, and verification utility** (this is NOT a sixth pipeline module and must not be referred to as "Module F").
- **Function:**
  - The final public release consists of Modules A through E combined with this release-preparation verification and split-correction step.
  - Enforces strict zero-leakage partitioning using scikit-learn `GroupShuffleSplit` (seed=42) keyed on the sample numeric identifier pair (pairing `benign_NNNNNNNN` with `malicious_NNNNNNNN`).
  - Guarantees that benign source documents and their corresponding paired adversarial variants are placed exclusively within the same split partition.
  - Reconciles and verifies the authoritative three Parquet split files:
    - `data/train.parquet` (231,900 rows, 80.00%)
    - `data/validation.parquet` (28,986 rows, 10.00%)
    - `data/test.parquet` (28,988 rows, 10.00%)

---

## 4. Authoritative Released Files

The files constituting the public release repository are strictly repository-relative:

```
PIID-PUBLIC-RELEASE/
├── data/
│   ├── train.parquet        (231,900 rows; 115,950 benign, 115,950 malicious)
│   ├── validation.parquet   ( 28,986 rows;  14,493 benign,  14,493 malicious)
│   ├── test.parquet         ( 28,988 rows;  14,494 benign,  14,494 malicious)
│   └── images/              (  1,870 physically included PNG files)
├── metadata/
│   ├── dataset_manifest.json (dataset parameters, authoritative counts, and split hashes)
│   ├── attack_taxonomy.json  (15 attack families and 58 payload specifications)
│   ├── image_manifest.csv    (catalog of the 1,870 released PNG images)
│   ├── sample_index.csv      (full index of 289,874 records)
│   └── validation_report.json(20-point programmatic integrity validation report)
├── docs/
│   ├── AUDIT_REPORT_v3.0.md
│   ├── DATASET_CARD.md
│   ├── DATA_PROVENANCE.md
│   ├── LICENSE_NOTES.md
│   └── PIPELINE_OVERVIEW.md
├── code/
│   ├── 01_Module_A_FINAL.ipynb
│   ├── 02_Module_B_FINAL.ipynb
│   ├── 03_Module_C_FINAL.ipynb
│   ├── 04_Module_D_FINAL.ipynb
│   ├── 05_Module_E_FINAL.ipynb
│   ├── module_f_step0_fix_splits.py
│   └── requirements.txt
├── CHECKSUMS.sha256
├── CITATION.cff
├── DATASET_CARD.md
├── FINAL_RELEASE_VALIDATION.md
├── LICENSE
└── README.md
```

---

## 5. Reproducibility & Determinism

| Parameter | Specification |
|---|---|
| Random seed | 42 |
| Split method | `GroupShuffleSplit` (scikit-learn) |
| Train split | 80% (231,900 records) |
| Validation split | 10% (28,986 records) |
| Test split | 10% (28,988 records) |
| Group key | Sample numeric identifier suffix |
| Determinism note | Text generation and split assignment are strictly deterministic under the documented seed (42) and configuration. OCR extraction and image rendering can exhibit environment-dependent variations across underlying library versions and system font renderers. |
| Python requirement | Python 3.10+ |

---

## 6. Pipeline Dependency Flow

```
01_Module_A ──► payload_bank.json
                      │
                      ▼
02_Module_B ──► malicious_text.csv
                      │
                      ▼
03_Module_C ──► visual image renderings & layout metadata
                      │
                      ▼
04_Module_D ──► validated malicious dataset partition
                      │
MASTER_BENIGN ────────┼──► 05_Module_E ──► consolidated benchmark
                                                  │
                                                  ▼
                                   module_f_step0_fix_splits.py (Utility)
                                                  │
                                                  ▼
                                      Authoritative Release Parquets
                                      + 1,870 Released Images
```

---

*Document prepared for the PIID PUBLIC RELEASE.*  
*Five-module pipeline architecture (A → B → C → D → E) is authoritative.*
