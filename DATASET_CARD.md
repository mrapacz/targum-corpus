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
size_categories:
- 10M<n<100M
task_categories:
- text-generation
- translation
---

# Targum Corpus

Targum is a multilingual New Testament translation corpus covering 657 translations across English, French, Spanish, Polish, and Italian, collected from 13 source libraries and spanning 1525–2025. This dataset contains the public release subset: 307 translations distributed under public domain or open licenses; the remaining 350 copyrighted translations are excluded.

## Dataset summary

| Language | Code | Translations |
|---|---|---|
| English | `en` | 196 |
| French | `fr` | 44 |
| Spanish | `es` | 29 |
| Polish | `pl` | 25 |
| Italian | `it` | 13 |
| **Total** | | **307** |

Includes public domain (242) and open-license (65) translations. Full translation metadata in `index.tsv` and `manifest.json`.

## Structure

```
corpora/
  {site}/
    {iso}/
      {id}.jsonl        # one verse per line
index.tsv               # metadata for all 307 translations
book_coverage.tsv       # which books each translation covers
manifest.json           # summary statistics
```

Each JSONL file:

```json
{"book": "MAT", "chapter": 1, "verse": "1", "text": "The book of the generation of Jesus Christ..."}
{"book": "MAT", "chapter": 1, "verse": "2", "text": "Abraham begat Isaac..."}
```

`book` uses standard 3-letter New Testament codes: `MAT MRK LUK JHN ACT ROM 1CO 2CO GAL EPH PHP COL 1TH 2TH 1TI 2TI TIT PHM HEB JAS 1PE 2PE 1JN 2JN 3JN JUD REV`.

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
# {'book': 'MAT', 'chapter': 1, 'verse': '1', 'text': 'This is the genealogy of Jesus...'}

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
    usecols=["translation_id", "translation_name", "iso", "canonical_year", "copyright_status"],
)
```

## Source data

Translations were scraped from 13 libraries: biblegateway.com, bible.com, biblehub.com, bible.is, ebible.org, bibles.org, bible.audio, jw.org, biblestudytools.com, bibliepolskie.pl, obohu.cz, crossbible.com, laparola.net.

Only public domain and open-license translations are included in this release. Copyrighted translations (350 of 657 total) are excluded.

## License

Corpus metadata and derived annotations: [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Individual translations retain their original licenses as recorded in `index.tsv`.

## Citation

Citation TBD. Preprint: [arxiv.org/abs/2602.09724](https://arxiv.org/abs/2602.09724)
