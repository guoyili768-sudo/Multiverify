# MultiVerify Dataset

> **Associated Paper:**
> Guoyi Li, Die Hu, Haozhe Li, Qirui Tang, Xiaomeng Fu, Yulei Wu, Xiaodan Zhang, and Honglei Lyu.
> *"Zero-Shot Multimodal Fact-Checking with Conceptual Reasoning."*
> In *Proceedings of the 33rd ACM International Conference on Multimedia (MM '25)*, October 27–31, 2025, Dublin, Ireland.
> DOI: [10.1145/3746027.3754836](https://doi.org/10.1145/3746027.3754836) 
---

## Overview

**MultiVerify** is a large-scale, multimodal, multi-domain fact-checking dataset for evaluating claim verification models in real-world everyday scenarios. It is built upon the [MultiFC](https://github.com/MultiFCDataset/MultiFC) dataset and extends it with:

- **Multimodal image evidence** — up to 10 image URLs retrieved per claim via Google Vision API
- **Rich text evidence** — full-length fact-checking articles and/or retrieved evidence snippets
- **Standardized binary labels** — all fine-grained verdicts are mapped to `Supported` (1) or `Refuted` (0)
- **Domain classification** — claims are categorized into 4 real-world domains using Google Content Classifier with manual review
- **Train / Dev / Test splits** — stratified 70/15/15 splits preserving per-domain label distribution

MultiVerify is designed to assess **cross-domain generalization** and **multimodal evidence integration** under realistic conditions with higher noise and evidence uncertainty than existing benchmarks.

---




## File Structure

```
MultiVerify_dataset/
├── claims.txt          Full claim list with all attributes
├── train.txt           Training split
├── dev.txt             Development split 
├── test.txt            Test split
├── evidence.txt        Text evidence per claim
├── source_index.txt    Index of claims grouped by fact-checking source
└── README.md           This file
```

---

## File Format

### `claims.txt` / `train.txt` / `dev.txt` / `test.txt`

One JSON object per line. Each record contains:

```json
{
  "id":           "pomt-09611",
  "split":        "train",
  "label":        1,
  "label_str":    "true",
  "domain":       "Politics",
  "source":       "PolitiFact",
  "text":         "No Democratic campaign for (Fla.) governor has ever had these kinds of resources this early on in an election cycle.",
  "image_urls":   ["https://...", "https://...", ...],
  "checker":      "ABC News",
  "tags":         "['elections', 'florida']",
  "article_title":"Alex Sink's fundraising claim",
  "publish_date": "2010-01-08",
  "claim_date":   "2010-01-06",
  "entities":     "['Florida', 'Alex Sink']",
  "text_source":  "factcheck_article"
}
```

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique claim identifier (source-prefix + index) |
| `split` | string | `train` / `dev` / `test` |
| `label` | int | `0` = Refuted, `1` = Supported |
| `label_str` | string | Original verdict text (e.g., `"mostly true"`, `"pants on fire!"`) |
| `domain` | string | One of 4 domains (see above) |
| `source` | string | Fact-checking outlet (e.g., PolitiFact, Snopes, GossipCop) |
| `text` | string | The claim to be verified |
| `image_urls` | list[str] | Retrieved image evidence URLs (up to 10) |
| `checker` | string | Tags/categories from the original outlet |
| `tags` | string | Topic tags |
| `article_title` | string | Title of the fact-checking article |
| `publish_date` | string | Date the fact-check was published |
| `claim_date` | string | Date the claim was made |
| `entities` | string | Named entities in the claim |
| `text_source` | string | `factcheck_article` / `multifc_evidence` / `semantic_rewrite` |

### `evidence.txt`

One JSON object per line, linked to claims via `ref_id`:

```json
{
  "id":          "0",
  "ref_id":      "pomt-09611",
  "text":        "Florida's leading Republican candidate for governor, Bill McCollum...",
  "text_source": "factcheck_article"
}
```

| Field | Description |
|---|---|
| `id` | Sequential evidence ID |
| `ref_id` | Corresponding claim ID |
| `text` | Full fact-checking article text or retrieved evidence snippets |
| `text_source` | `factcheck_article`: full article from fact-checking outlet; `multifc_evidence`: retrieved snippets from MultiFC; `semantic_rewrite`: augmented via semantic rewriting |

### `source_index.txt`


---




## Label Definition

| Label | Value | Description |
|---|---|---|
| Supported | `1` | The claim is factually accurate based on available evidence |
| Refuted | `0` | The claim is false, misleading, or unverified based on available evidence |

All fine-grained verdicts from original outlets (e.g., `"mostly true"`, `"half-true"`, `"pants on fire!"`, `"fiction!"`) are mapped to this binary scheme following the convention in the associated paper: `"True"` label → **Supported**; all others → **Refuted**.

---

## Text Evidence Sources

| Source | Count | Description |
|---|---|---|
| `factcheck_article` | 9,942 | Full article text scraped from fact-checking websites |
| `multifc_evidence` | 8,871 | Retrieved evidence snippets from MultiFC (avg. 9.2 snippets/claim) |
| `semantic_rewrite` | 1,286 | Augmented records generated via LLM-based semantic rewriting for class balance |

---

## Usage Example

```python
import json

# Load all claims
claims = {}
with open('claims.txt') as f:
    for line in f:
        d = json.loads(line)
        claims[d['id']] = d

# Load evidence
evidence = {}
with open('evidence.txt') as f:
    for line in f:
        d = json.loads(line)
        evidence[d['ref_id']] = d['text']

# Load train split
train = []
with open('train.txt') as f:
    for line in f:
        train.append(json.loads(line))

print(f"Train: {len(train)} | Claims: {len(claims)} | Evidence: {len(evidence)}")

# Access a sample
sample = train[0]
print(f"Claim : {sample['text']}")
print(f"Label : {'Supported' if sample['label'] == 1 else 'Refuted'}")
print(f"Domain: {sample['domain']}")
print(f"Images: {len(sample['image_urls'])} URLs")
print(f"Evidence: {evidence.get(sample['id'], '')[:200]}")
```

---




## Citation

If you use this dataset, please cite:

```bibtex
@inproceedings{li2025zero,
  title={Zero-Shot Multimodal Fact-Checking with Conceptual Reasoning},
  author={Li, Guoyi and Hu, Die and Li, Haozhe and Tang, Qirui and Fu, Xiaomeng and Wu, Yulei and Zhang, Xiaodan and Lyu, Honglei},
  booktitle={Proceedings of the 33rd ACM International Conference on Multimedia},
  pages={62--71},
  year={2025}
}
```

Also cite the original MultiFC dataset:

```bibtex
@inproceedings{augenstein2019multifc,
  title     = {MultiFC: A Real-World Multi-Domain Dataset for Evidence-Based Fact Checking of Claims},
  author    = {Isabelle Augenstein and Christina Lioma and Dongsheng Wang and Lucas Chaves Lima and Casper Hansen and Christian Hansen and Jakob Grue Simonsen},
  booktitle = {Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing (EMNLP)},
  year      = {2019}
}
```

---

## License

This dataset is released under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/), consistent with the associated paper. Claims are sourced from publicly available fact-checking websites; image URLs point to publicly accessible resources.

