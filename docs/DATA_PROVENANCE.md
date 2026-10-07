# DATA PROVENANCE — PIID Dataset
**Version:** 1.0.0 | **Pipeline Architecture:** Five-Module (A → B → C → D → E) + Release Verification  
**Target Repository:** Mendeley Data

---

## 1. Overview & Dataset Accounting

The PIID dataset is an empirical insurance-domain prompt-injection detection benchmark for AI workflows. The release is predominantly text-based, with 1,870 records retaining associated document images, combining textual and visual contexts derived from publicly available document corpora with fully synthetic adversarial injections engineered through a five-module generation pipeline.

- **Total Benchmark Records:** 289,874 (144,937 benign | 144,937 malicious)
- **Partitions & Split Methodology:** Partitioned via scikit-learn `GroupShuffleSplit` with seed 42 across an identical 9-column schema (`sample_id`, `text`, `ocr_text`, `image_path`, `label`, `attack_family`, `payload_id`, `source_module`, `split`).
  - `data/train.parquet`: 231,900 records (115,950 benign, 115,950 malicious)
  - `data/validation.parquet`: 28,986 records (14,493 benign, 14,493 malicious)
  - `data/test.parquet`: 28,988 records (14,494 benign, 14,494 malicious)
- **Pipeline Architecture & Utilities:** Constructed via the authoritative five-module pipeline (A → B → C → D → E) covering 15 attack families and 58 payloads. `code/module_f_step0_fix_splits.py` is a release-preparation and split-correction utility, not a sixth pipeline module ("Module F").
- **Modality Accounting (Released Artifact):**
  - **Released Text-Only Records:** **288,004** (99.355%; `image_path` is null)
  - **Released Records with Images:** **1,870** (0.645%; `image_path` references physical PNGs in `data/images/`)
- **Image Artifact Accounting:**
  - **3,687** image artifacts were generated during development / pipeline execution.
  - **1,870** images were retained and are physically included in the public release under `data/images/`.
  - **1,817** historical image references were not retained in local storage and were sanitized to `null` during release packaging (no broken or dangling release image paths exist). Their textual content and OCR text are preserved.
  - **Zero** broken or dangling image paths exist in the released dataset.

**Explicit Distribution Notice:**
> **No original, unmodified third-party image files or raw source corpora are distributed in this release.**  
> Retained PNG images in `data/images/` are project-generated composite artifacts produced by rendering adversarial injections over document backgrounds. Underlying third-party background rights and restrictions remain applicable as detailed below.

---

## 2. Upstream Source Provenance

### Source 1 — CORD (Consolidated Receipt Dataset)

| Field | Detail |
|---|---|
| **Dataset Name** | CORD: A Consolidated Receipt Dataset for Post-OCR Parsing |
| **Upstream Corpus** | CORD Benchmark Dataset |
| **Role in Pipeline** | Benign document text source — OCR-extracted textual content utilized as benign domain text in Module A / Module E |
| **License** | Creative Commons Attribution 4.0 International (CC BY 4.0) |
| **Redistribution** | Permitted under CC BY 4.0 with attribution |
| **Attribution** | Park, S., Shin, S., Baek, J., et al. (2019). *CORD: A Consolidated Receipt Dataset for Post-OCR Parsing*. NeurIPS 2019 Workshop on Document Intelligence. |
| **What is Released** | Derived textual content (post-OCR extraction); no original CORD file structures or images are distributed |

---

### Source 2 — DocVQA

| Field | Detail |
|---|---|
| **Dataset Name** | DocVQA: A Dataset for VQA on Document Images |
| **Upstream Corpus** | DocVQA Document Image Corpus |
| **Role in Pipeline** | Benign document background source utilized by Module C during image synthesis |
| **License** | Research use only — see terms at https://www.docvqa.org |
| **Redistribution Notice** | **Original images are NOT redistributed.** Retained PNG files in `data/images/` are project-generated composite artifacts that may incorporate DocVQA-derived backgrounds. Applicable upstream restrictions remain relevant. |
| **User Obligation** | Users must independently evaluate compatibility with DocVQA terms before any commercial use or downstream redistribution |
| **Attribution** | Mathew, M., Karatzas, D., & Jawahar, C. V. (2021). *DocVQA: A dataset for VQA on document images*. IEEE/CVF Winter Conference on Applications of Computer Vision (WACV), 2021. |

---

### Source 3 — LIC India Official Forms

| Field | Detail |
|---|---|
| **Dataset Name** | Life Insurance Corporation of India — Public Regulatory Claim & Policy Forms |
| **Upstream Source** | Publicly accessible policy documentation and regulatory claim forms from licindia.in |
| **Role in Pipeline** | Benign insurance domain document structure and textual source |
| **License / Terms** | **Publicly accessible source material.** Do not assume public domain status. Redistribution and derivative-use conditions must be evaluated under applicable upstream terms. |
| **User Obligation** | Users should consult official terms at licindia.in prior to redistribution or commercial deployment |
| **What is Released** | Derived text extracts and rendered composite document page instances |

---

### Source 4 — NAIC Model Laws & Regulations

| Field | Detail |
|---|---|
| **Dataset Name** | National Association of Insurance Commissioners (NAIC) Model Laws, Regulations, and Guidelines |
| **Upstream Source** | NAIC public regulatory publications (naic.org) |
| **Role in Pipeline** | Benign insurance regulatory and statutory text source |
| **License / Terms** | Publicly accessible policy and regulatory reference documents. Public accessibility does not automatically imply unrestricted redistribution. Attribution required; applicable upstream terms should be checked. |
| **Attribution** | © National Association of Insurance Commissioners (NAIC). Used for academic research and educational evaluation. |
| **User Obligation** | Verify current NAIC terms and conditions at naic.org |
| **What is Released** | Derived textual extracts used for benign insurance classification benchmarks |

---

### Source 5 — RVL-CDIP

| Field | Detail |
|---|---|
| **Dataset Name** | RVL-CDIP Document Image Dataset |
| **Upstream Corpus** | RVL-CDIP 400,000 Scanned Document Corpus |
| **Role in Pipeline** | Scanned document backgrounds utilized by Module C during composite image synthesis |
| **License** | **Non-commercial academic research only.** |
| **Redistribution Notice** | **Original images are NOT redistributed.** Retained PNG files in `data/images/` are project-generated composite artifacts that may incorporate RVL-CDIP backgrounds. Upstream non-commercial academic research restrictions remain applicable. |
| **User Obligation** | Commercial deployment of composite images incorporating RVL-CDIP backgrounds is NOT permitted without an independent license from the copyright holders. See https://www.cs.cmu.edu/~aharley/rvl-cdip/ |
| **Attribution** | Harley, A. W., Ufkes, A., & Derpanis, K. G. (2015). *Evaluation of deep convolutional nets for document image classification and retrieval*. International Conference on Document Analysis and Recognition (ICDAR), 2015. |

---

### Source 6 — Synthetically Generated Adversarial Injections (Modules B, C, D)

| Field | Detail |
|---|---|
| **Description** | All 144,937 malicious records were constructed by injecting 58 synthetic prompt-injection payload templates across 15 attack families into benign insurance contexts using Modules B and C |
| **Role in Pipeline** | Primary source of all adversarial prompt-injection content across text and multimodal channels |
| **License** | **CC BY 4.0** applies strictly to the project-created synthetic payload templates and project-authored injection text for which the project holds rights |
| **Crucial Clarification** | The application of a synthetic injection does NOT cause underlying third-party source material to become CC BY 4.0. Underlying benign text and visual backgrounds remain subject to their respective upstream rights and restrictions |
| **Safety Notice** | Adversarial payload templates are purely synthetic academic constructs designed to test LLM safety guards. They contain no real-world exploit code, private credentials, or live PII |

---

## 3. Summary of Released Material & Upstream Terms

| Source | Contributed Material | Upstream Term / Status | Released Derivative | Upstream Restrictions Apply? |
|---|---|---|---|---|
| CORD | Benign receipt / invoice text | CC BY 4.0 | OCR-extracted text | Yes (CC BY 4.0 Attribution) |
| DocVQA | Visual document backgrounds | Research use only | Composite rendered PNGs | Yes (Research use only) |
| LIC India | Insurance claim forms text/layouts | Publicly accessible material | Derived text & rendered pages | Yes (Verify terms at licindia.in) |
| NAIC | Regulatory statutes / model laws | Public regulatory reference | Extracted domain text | Yes (Attribution required) |
| RVL-CDIP | Document page backgrounds | Non-commercial academic | Composite rendered PNGs | Yes (Non-commercial academic only) |
| Synthetic (B/C/D) | 144,937 injection texts & layouts | Project-authored | Text & visual injections | Project CC BY 4.0 applies to synthetic parts |

---

*This provenance record reflects the verified contents of the PIID PUBLIC RELEASE.*  
*Consult `docs/LICENSE_NOTES.md` for complete composite licensing guidance.*
