---
language:
- en
- fr
- es
- pl
- it
license: cc-by-4.0
pretty_name: Targum Corpus
tags:
- bible
- translation
- nlp
- corpus-linguistics
- parallel-corpus
- digital-humanities
size_categories:
- 10M<n<100M
task_categories:
- text-generation
- translation
configs:
- config_name: index
  data_files: "index.tsv"
  default: true
- config_name: book_coverage
  data_files: "book_coverage.tsv"
- config_name: copyrights
  data_files: "copyrights.tsv"
---

# Targum Corpus

**Targum** is a multilingual New Testament translation corpus with unprecedented depth in five European languages: English, French, Italian, Polish, and Spanish. It contains **657** translations (**349** unique) collected from **13** source libraries and spanning **1525–2025**.

This dataset contains the **public release subset**: **307** translations distributed under public domain or open licenses. The remaining 350 copyrighted translations are not distributed but are available to researchers upon request.

Named after the ancient Aramaic translations of the Hebrew Bible (תרגום, "translation"), the corpus prioritizes vertical depth over linguistic breadth, making it possible to computationally analyze a wide spectrum of historical periods and confessional traditions within each language.

Also available on GitHub: [mrapacz/targum-corpus](https://github.com/mrapacz/targum-corpus).

## Corpus Scale

| Language | Code | Total | Unique | Public subset |
|---|---|---:|---:|---:|
| English | `eng` | 396 | 208 | 196 |
| French | `fra` | 78 | 41 | 44 |
| Spanish | `spa` | 102 | 54 | 29 |
| Polish | `pol` | 48 | 29 | 25 |
| Italian | `ita` | 33 | 17 | 13 |
| **Total** | | **657** | **349** | **307** |

"Total" is the number of translation instances collected across all 13 source libraries (the same translation may appear on multiple sites). "Unique" is the number of distinct translation editions after deduplication. "Public subset" is the number of instances distributed in this dataset under public domain (242) or open licenses (65).

## Structure

```
corpora/
  {site}/
    {iso}/
      {id}.jsonl        # one verse per line
index.tsv               # metadata for all 657 translations
copyrights.tsv          # copyright text and status per translation
book_coverage.tsv       # which books each translation covers
manifest.json           # summary statistics
```

Each JSONL file:

```json
{"book": "MAT", "chapter": 1, "verse": "1", "text": "The book of the generation of Jesus Christ..."}
{"book": "MAT", "chapter": 1, "verse": "2", "text": "Abraham begat Isaac..."}
```

`book` uses USFM 3-letter New Testament codes: `MAT MRK LUK JHN ACT ROM 1CO 2CO GAL EPH PHP COL 1TH 2TH 1TI 2TI TIT PHM HEB JAS 1PE 2PE 1JN 2JN 3JN JUD REV`.

## Metadata

Each translation in `index.tsv` is annotated with manually verified metadata:

- **`canonical_id`** — a standardized identifier for the translation work (e.g. `kjv`, `nkjv`). Any version presented under a new name receives its own unique identifier.
- **`canonical_version`** — the specific edition or revision (e.g. `1611`, `4th edition`).
- **`canonical_year`** — the year of the specific revision.
- **`copyright_status`** — one of `public_domain`, `open_license`, or `copyrighted`.

This canonicalization allows researchers to define "uniqueness" for their own needs: they can perform micro-level analyses on translation families (e.g. the KJV lineage) or conduct macro-level studies by deduplicating closely related texts.

## Usage

```python
from datasets import load_dataset

# Load a single translation
ds = load_dataset(
    "mrapacz/targum-corpus",
    data_files="corpora/ebible.org/eng/eng-web.jsonl",
    split="train",
)
print(ds[0])

# Load all English translations from one site
ds = load_dataset(
    "mrapacz/targum-corpus",
    data_files="corpora/ebible.org/eng/*.jsonl",
    split="train",
)
```

Load metadata:

```python
import pandas as pd

index = pd.read_csv(
    "hf://datasets/mrapacz/targum-corpus/index.tsv",
    sep="\t",
)
```

## Source Data

Translations were collected from 13 libraries: bible.audio, bible.com, bible.is, biblegateway.com, biblehub.com, bibles.org, biblestudytools.com, bibliepolskie.pl, crossbible.com, ebible.org, jw.org, laparola.net, obohu.cz.

## Citation

If you use this corpus, please cite:

```bibtex
@inproceedings{rapacz2026targum,
  title={ {Targum} -- A Multilingual New Testament Translation Corpus},
  author={Rapacz, Maciej and Smywi{\'n}ski-Pohl, Aleksander},
  booktitle={Proceedings of the 2026 Joint International Conference on Computational Linguistics, Language Resources and Evaluation (LREC-COLING 2026)},
  year={2026}
}
```

Preprint: [arxiv.org/abs/2602.09724](https://arxiv.org/abs/2602.09724)

## License

Corpus metadata and derived annotations: [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Individual translations retain their original licenses as recorded in `copyrights.tsv`.
