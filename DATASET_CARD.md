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

![Targum — A Multilingual New Testament Translation Corpus](banner.jpg)

**Targum** is a multilingual New Testament translation corpus with unprecedented depth in five European languages: English, French, Italian, Polish, and Spanish. It contains **651** translations (**334** unique) collected from **13** source libraries and spanning **1525–2025**.

This dataset contains the **public release subset**: **302** translations distributed under public domain or open licenses. The remaining 349 copyrighted translations are not distributed but are available to researchers upon request.

Named after the ancient Aramaic translations of the Hebrew Bible (תרגום, "translation"), the corpus prioritizes vertical depth over linguistic breadth, making it possible to computationally analyze a wide spectrum of historical periods and confessional traditions within each language.

Also available on GitHub: [mrapacz/targum-corpus](https://github.com/mrapacz/targum-corpus).

## Corpus Scale

| Language | Code | Total | Unique | Public subset |
|---|---|---:|---:|---:|
| English | `eng` | 390 | 194 | 191 |
| French | `fra` | 78 | 41 | 44 |
| Spanish | `spa` | 102 | 53 | 29 |
| Polish | `pol` | 48 | 29 | 25 |
| Italian | `ita` | 33 | 17 | 13 |
| **Total** | | **651** | **334** | **302** |

"Total" is the number of translation instances collected across all 13 source libraries (the same translation may appear on multiple sites). "Unique" is the number of distinct translation editions after deduplication. "Public subset" is the number of instances distributed in this dataset under public domain (237) or open licenses (65).

## Structure

```
corpora/
  {site}/
    {iso}/
      {id}.jsonl        # one verse per line
index.tsv               # metadata for all 651 translations
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

## Embeddings

Pre-computed text embeddings are available for all translations at both chapter and verse granularity, produced by two Qwen3 embedding models:

| Model | Granularity | Files | Size |
|---|---|---:|---:|
| `Qwen/Qwen3-Embedding-0.6B` | chapter | 656 | ~270 MB |
| `Qwen/Qwen3-Embedding-0.6B` | verse | 656 | ~7.7 GB |
| `Qwen/Qwen3-Embedding-8B` | chapter | 656 | ~2.4 GB |
| `Qwen/Qwen3-Embedding-8B` | verse | 656 | ~75 GB |

Embeddings are stored as Hive-partitioned Parquet files:

```
embeddings/
  {model}/
    language={iso}/
      site={site}/
        translation={id}/
          granularity={chapter|verse}/
            data.parquet
```

Where `{model}` uses `XxX` as a separator (e.g. `QwenXxXQwen3-Embedding-0.6B`). Each `data.parquet` contains columns `key` (e.g. `MAT 1` for chapter, `MAT 1:1` for verse) and `value` (the embedding vector).

Loading chapter embeddings for one translation:

```python
import pandas as pd

df = pd.read_parquet(
    "hf://datasets/mrapacz/targum-corpus/embeddings/"
    "QwenXxXQwen3-Embedding-0.6B/language=eng/site=ebible.org/"
    "translation=eng-web/granularity=chapter/data.parquet"
)
```

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

This corpus will be published at [LREC 2026](https://lrec2026.info/) (Palma, Mallorca, 11--16 May 2026). Citation TBD.

A preprint is available at [arxiv.org/abs/2602.09724](https://arxiv.org/abs/2602.09724).

## License

Corpus metadata and derived annotations: [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Individual translations retain their original licenses as recorded in `copyrights.tsv`.
