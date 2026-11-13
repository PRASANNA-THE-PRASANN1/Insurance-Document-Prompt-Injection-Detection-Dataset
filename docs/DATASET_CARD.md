---
annotations_creators:
- machine-generated
language_creators:
- machine-generated
language:
- en
license:
- cc-by-4.0
- other-restrictions-apply
multilinguality:
- monolingual
size_categories:
- 100K<n<1M
source_datasets:
- CORD
- DocVQA
- LIC
- NAIC
- RVL-CDIP
- synthetic
task_categories:
- text-classification
task_ids:
- binary-classification
---

# Dataset Card — PIID

## Dataset Description

- **Repository:** PIID-PUBLIC-RELEASE/ ([GitHub](https://github.com/PRASANNA-THE-PRASANN1/Insurance-Document-Prompt-Injection-Detection-Dataset))
- **Permanent Archive:** Mendeley Data ([DOI: 10.17632/5b6xz84n79.1](https://data.mendeley.com/datasets/5b6xz84n79/1))
- **Version:** 1.0.0 (Five-Module Pipeline: A → B → C → D → E + Release Verification)
- **License:** Composite licensing model (CC BY 4.0 for project-created materials; upstream terms apply to third-party-derived components; see `docs/LICENSE_NOTES.md`)

### Dataset Summary

PIID is a large-scale insurance-domain prompt-injection detection benchmark constructed for AI security. The dataset comprises **289,874** standardized samples (balanced: 144,937 benign and 144,937 malicious). The release is predominantly text-based, with 1,870 records retaining associated document images, incorporating **15 adversarial attack families** and **58 unique prompt-injection payload templates**.

The benchmark was constructed using a five-module generation pipeline (A → B → C → D → E) followed by a group-leakage-free split verification step with random seed 42.

### Supported Tasks

- **Primary Task — Binary Prompt-Injection Detection:** Classifying document text (and associated document images where available) as `"benign"` or `"malicious"`.
- **Secondary Task — Attack-Family Attribution:** Fine-grained categorization of malicious instances among the 15 attack families (benign instances carry `"none"`). The dataset does not present 15 mutually exclusive classes for all records; rather, attack family attribution applies to the malicious subset.
- **Document Image Prompt-Injection Robustness:** Evaluating vision-language model robustness against adversarial instructions visually rendered into document page images across the retained 1,870 image records.

### Languages

English (`en`), monolingual insurance and legal domain documentation.

---

## Dataset Structure

### Data Instances

Each sample is contained in `data/train.parquet`, `data/validation.parquet`, or `data/test.parquet`, sharing an identical 9-column schema:

```python
{
    "sample_id":     "malicious_00012345",
    "text":          "Claim Form #40291... [INJECTED PROMPT]",
    "ocr_text":      "Claim Form #40291...",
    "image_path":    "data/images/00112154ee1e4698.png",  # or None / null for text-only
    "label":         "malicious",
    "attack_family": "Context Poisoning",
    "payload_id":    "PAY-CP-89d53fcfa6",
    "source_module": "MASTER_MALICIOUS_DATASET",
    "split":         "train"
}
```

### Data Fields

| Field | Type | Description |
|---|---|---|
| `sample_id` | string | Canonical record identifier (`benign_NNNNNNNN` / `malicious_NNNNNNNN`) |
| `text` | string | Full textual content of document or prompt-injected text |
| `ocr_text` | string | OCR-extracted text (populated for image records; duplicated/empty as applicable) |
| `image_path` | string or null | Relative file path to image in `data/images/`; `null` for text-only records |
| `label` | string | Ground-truth class: `"benign"` or `"malicious"` |
| `attack_family` | string | Attack category (15 families for malicious; `"none"` for benign) |
| `payload_id` | string | Specific payload template identifier (`PAY-*` for malicious; `"none"` for benign) |
| `source_module` | string | Origin module tag (`MASTER_BENIGN_DATASET` or `MASTER_MALICIOUS_DATASET`) |
| `split` | string | Split partition tag (`"train"`, `"validation"`, or `"test"`) |

### Data Splits & Modality Breakdown

Splits were partitioned using scikit-learn `GroupShuffleSplit` (seed=42) keyed on the sample numeric ID pair, ensuring benign and malicious variants of the same source pair remain strictly within the same partition:

| Split | Total Rows | Benign | Malicious | % | Records with Images | Text-Only Records |
|---|---|---|---|---|---|---|
| `train.parquet` | 231,900 | 115,950 | 115,950 | 80.000% | 1,504 | 230,396 |
| `validation.parquet` | 28,986 | 14,493 | 14,493 | 10.000% | 184 | 28,802 |
| `test.parquet` | 28,988 | 14,494 | 14,494 | 10.000% | 182 | 28,806 |
| **Total** | **289,874** | **144,937** | **144,937** | **100.000%** | **1,870** | **288,004** |

**Image & Modality Accounting:**
- **Released records with images = 1,870:** Exactly 1,870 records have physically available PNG image files in `data/images/`.
- **Released text-only records = 288,004:** Exactly 288,004 records have `image_path = null`.
- The 1,817 historical image references are NOT active image samples in the public release because their images are not distributed and their `image_path` values are `null`.

---

## Data Quality & Integrity

Programmatic validation over the released repository confirms:
- **289,874** unique sample IDs.
- **Zero** duplicate sample IDs.
- **Zero** cross-split sample ID overlap.
- **Zero** paired-source / group leakage according to the release audit.
- **Identical 9-column schema** across all three Parquet splits.
- **Zero** broken or dangling image paths in the public release.
- **1,870 physical PNG files** are included in `data/images/`.

---

## Dataset Creation

### Curation Rationale

Prompt injection against LLMs embedded in claims processing, underwriting, and policy customer service can lead to automated unauthorized payouts, policy bypasses, and data exfiltration. PIID provides an empirical benchmark to test detection defenses across text and visual document channels.

### Source Data

- **Benign corpora:** Derived textual content and document page contexts originating from CORD, DocVQA, LIC India public forms, NAIC model regulations, and RVL-CDIP.
- **Adversarial injections:** Synthetically engineered by Modules B and C covering 15 attack families (Context Poisoning, Instruction Override, Footer Injection, Policy Bypass, White Text Injection, Metadata Injection, Insurance-specific Attacks, Hidden Multimodal, Header Injection, Data Exfiltration, Decision Manipulation, OCR-specific Attacks, Invisible Character Injection, Tiny Font Injection, Watermark Injection).

---

## Limitations

- **Modality Distribution:** The benchmark is predominantly text-based (288,004 text-only samples / 99.355%). Records retaining associated document images represent 0.645% (1,870 records).
- **Image Limitation:** Module C generated 3,687 image artifacts during development. 1,870 are retained in the public release. The remaining 1,817 historical image references were sanitized to null during release packaging. No broken or dangling image paths remain in the released dataset.
- **Third-Party Background Restrictions:** Composite images in `data/images/` may incorporate background material derived from DocVQA and RVL-CDIP, which carry non-commercial and research-only restrictions. See `docs/LICENSE_NOTES.md`.

---

## Licensing Information

The dataset uses a **composite licensing model**:
- **Project-created synthetic components** (including synthetic adversarial payloads, injected text, and project-authored metadata, documentation, and code) are licensed by the project under **Creative Commons Attribution 4.0 International (CC BY 4.0)** where the project owns the applicable rights.
- **Third-party-derived material** remains subject to the applicable upstream source terms.
- The project’s CC BY 4.0 statement **does not override upstream restrictions**.
- Users must consult `docs/LICENSE_NOTES.md` before redistribution or commercial use.

---

## Citation Information

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
  note         = {Five-module synthetic pipeline. 289,874 samples, 15 attack families, 58 payloads.}
}
```
*(Deposited on Mendeley Data with permanent DOI: `10.17632/5b6xz84n79.1`)*
