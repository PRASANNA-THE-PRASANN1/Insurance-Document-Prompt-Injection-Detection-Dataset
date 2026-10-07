# License Notes & Terms of Use — PIID Dataset
**Version:** 1.0.0 | **Date:** 2026-10-01  
**Target Repository:** Mendeley Data

---

## Overview: Layered & Composite Licensing Model

The PIID benchmark is distributed under a **layered, composite licensing framework**. Because the dataset combines newly authored synthetic adversarial content with material derived from multiple third-party document corpora, a single blanket license cannot apply universally to every component.

**The project does NOT state that the PIID dataset as a whole is unconditionally CC BY 4.0.** The project's grant of CC BY 4.0 applies strictly to project-owned and project-created materials, and does NOT override, modify, or extinguish any preexisting third-party rights or upstream source terms.

---

## Section 1 — Project-Created Material (CC BY 4.0)

For all components created directly by the project authors and for which the project holds the applicable copyright, the material is licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)** (https://creativecommons.org/licenses/by/4.0/).

This includes:
- **Synthetic adversarial payload templates:** The 58 prompt-injection payloads across the 15 attack families in `metadata/attack_taxonomy.json` and pipeline configurations.
- **Project-generated injection text:** Adversarial prompt-injection strings constructed in Module B.
- **Project-authored metadata and manifests:** `metadata/dataset_manifest.json`, `metadata/image_manifest.csv`, `metadata/validation_report.json`, and `metadata/sample_index.csv`.
- **Project-authored documentation:** All files in `docs/`, `README.md`, `DATASET_CARD.md`, `FINAL_RELEASE_VALIDATION.md`, and `CITATION.cff`.
- **Project-authored pipeline code & utilities:** Notebooks in `code/` and `code/module_f_step0_fix_splits.py`.
- Other project-created components for which the project owns the relevant intellectual property rights.

---

## Section 2 — Third-Party-Derived Material

Third-party-derived textual content and document page contexts remain subject to the applicable licenses and terms established by their original authors and rights holders:

1. **CORD (Consolidated Receipt Dataset):**
   - Derived OCR text extracted from receipts and invoices is released under **CC BY 4.0** per the upstream CORD license, with attribution required.
2. **DocVQA:**
   - Upstream source terms restrict usage to **non-commercial research only**. Derived content incorporated into composite renderings remains subject to DocVQA terms.
3. **LIC India Claim & Policy Forms:**
   - Derived from **publicly accessible regulatory forms** at licindia.in. Public accessibility does not establish public domain dedication. Redistribution and derivative use conditions must be interpreted per applicable upstream source terms.
4. **NAIC Model Laws and Regulations:**
   - Publicly accessible policy and regulatory reference documents. Usage requires formal copyright attribution (© National Association of Insurance Commissioners). Upstream terms must be verified at naic.org.
5. **RVL-CDIP Document Image Dataset:**
   - Scanned document backgrounds are restricted strictly to **non-commercial academic research only**.

**The project's CC BY 4.0 license does NOT override, supersede, or waive any of these upstream restrictions.**

---

## Section 3 — Composite Image Artifacts

The PNG files located in `data/images/` (1,870 files) are **project-generated composite artifacts**. They were produced by rendering synthetic prompt-injection payloads and typography over document background images.

> **“The PNG files in data/images/ are project-generated composite artifacts. They may incorporate backgrounds or source material derived from third-party corpora. The project does not grant a CC BY 4.0 license that overrides applicable upstream rights or restrictions.”**

Downstream users processing visual artifacts from `data/images/` must respect the underlying upstream restrictions (specifically DocVQA research-only and RVL-CDIP non-commercial research terms).

---

## Section 4 — Commercial Use Guidance

The project authors **do not make a blanket claim that all text-only data or all dataset components are automatically cleared for commercial use.**

Because different samples derive their benign document context from different upstream sources (e.g. CORD vs. LIC India vs. NAIC), downstream users and organizations are solely responsible for evaluating the provenance, licensing chain, and applicable upstream terms of the specific material, subsets, or modalities they intend to utilize.

For commercial AI applications:
- Users should examine the `source_module` and provenance documentation to determine whether specific records incorporate third-party sources with non-commercial or proprietary clauses.
- Samples with associated images incorporating DocVQA or RVL-CDIP backgrounds must not be used in commercial models without independent licenses from respective copyright holders.

---

## Section 5 — Required Citations & Attributions

Any research, publication, software, or evaluation utilizing the PIID dataset should provide attribution where required by the applicable license/terms:

### Project Citation
```bibtex
@dataset{piid_2026,
  title     = {Insurance Document Prompt-Injection Detection Dataset},
  year      = {2026},
  version   = {1.0.0},
  note      = {Five-module synthetic pipeline. 289,874 samples, 15 attack families, 58 payloads.},
}
```

### Upstream Sources Attribution

1. **CORD:**
   > Park, S., Shin, S., Baek, J., et al. (2019). *CORD: A Consolidated Receipt Dataset for Post-OCR Parsing*. NeurIPS 2019 Workshop on Document Intelligence.
2. **DocVQA:**
   > Mathew, M., Karatzas, D., & Jawahar, C. V. (2021). *DocVQA: A dataset for VQA on document images*. IEEE/CVF Winter Conference on Applications of Computer Vision (WACV), 2021. https://www.docvqa.org
3. **NAIC Model Laws:**
   > © National Association of Insurance Commissioners (NAIC). Used for academic and security research purposes. https://www.naic.org
4. **RVL-CDIP:**
   > Harley, A. W., Ufkes, A., & Derpanis, K. G. (2015). *Evaluation of deep convolutional nets for document image classification and retrieval*. International Conference on Document Analysis and Recognition (ICDAR), 2015. https://www.cs.cmu.edu/~aharley/rvl-cdip/
5. **LIC India Forms:**
   > Life Insurance Corporation of India (LIC). Publicly accessible regulatory forms and claims documentation. https://licindia.in
