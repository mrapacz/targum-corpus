# targum-corpus

Multilingual New Testament translation corpus. 307 translations across 5 languages from 13 source libraries, spanning 1525–2025.

## Contents

| Language | Code | Translations |
|---|---|---|
| English | `eng` | 196 |
| French | `fra` | 44 |
| Spanish | `spa` | 29 |
| Polish | `pol` | 25 |
| Italian | `ita` | 13 |
| **Total** | | **307** |

Includes public domain (242) and open-license (65) translations from biblegateway.com, bible.com, ebible.org, and 10 other libraries. Full metadata in [`manifest.json`](manifest.json) and [`index.tsv`](index.tsv).

## Structure

```
corpus/
  {site}/
    {iso}/
      {id}.jsonl     # one verse per line
index.tsv            # metadata for all 307 translations
book_coverage.tsv    # which books each translation covers
manifest.json        # summary statistics
```

Each JSONL file contains one verse per line:

```json
{"book": "MAT", "chapter": 1, "verse": "1", "text": "The book of the generation of Jesus Christ..."}
{"book": "MAT", "chapter": 1, "verse": "2", "text": "Abraham begat Isaac..."}
```

`book` uses standard 3-letter New Testament codes: `MAT MRK LUK JHN ACT ROM 1CO 2CO GAL EPH PHP COL 1TH 2TH 1TI 2TI TIT PHM HEB JAS 1PE 2PE 1JN 2JN 3JN JUD REV`.

## Quickstart

```python
import json
from pathlib import Path

corpus = Path("corpus")
# load one translation
verses = [json.loads(line) for line in (corpus / "ebible.org/eng/eng-web.jsonl").read_text().splitlines()]
print(f"{len(verses)} verses loaded")
# → 7957 verses loaded

# load all English translations
translations = {}
for path in sorted(corpus.glob("*/eng/*.jsonl")):
    translation_id = path.stem
    translations[translation_id] = [json.loads(line) for line in path.read_text().splitlines()]
print(f"{len(translations)} English translations")
```

Or via HuggingFace datasets:

```python
from datasets import load_dataset

ds = load_dataset("mrapacz/targum-corpus", data_files="corpus/ebible.org/eng/eng-web.jsonl", split="train")
```

## Index

`index.tsv` contains one row per translation with fields:
`translation_id`, `translation_name`, `translation_abbr`, `site`, `iso`, `num_books`, `num_chapters`, `num_verses`, `num_words`, `copyright`, `copyright_status`, `canonical_id`, `canonical_year`

## Citation

If you use this corpus, please cite:

```bibtex
@dataset{targum2026,
  author    = {Rapacz, Maciej},
  title     = {Targum Corpus: A Multilingual New Testament Translation Corpus},
  year      = {2026},
  url       = {https://github.com/mrapacz/targum-corpus},
}
```

## License

Corpus metadata and our derived annotations are released under [CC-BY 4.0](LICENSE).
Individual translations retain their original licenses as recorded in `index.tsv`.
Public domain translations are freely usable; open-license translations carry their respective terms.
