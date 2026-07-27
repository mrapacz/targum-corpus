# Targum -- A Multilingual New Testament Translation Corpus

**Links:** 📄 [LREC 2026 paper](https://aclanthology.org/2026.lrec-main.564/) · 🤗 [HuggingFace dataset](https://huggingface.co/datasets/mrapacz/targum-corpus) · 🌐 [Online viewer](https://targum.net/) · 🧪 [SIGHUM companion code](https://github.com/mrapacz/sighum-interlinear-vector-baselines)

**Targum** is a multilingual New Testament translation corpus with unprecedented depth in five European languages: English, French, Italian, Polish, and Spanish. It contains **651** translations (**334** unique) collected from **13** source libraries and spanning **1525–2025**.

This repository contains the **public release subset**: **301** translations distributed under public domain or open licenses.

Named after the ancient Aramaic translations of the Hebrew Bible (תרגום, "translation"), the corpus prioritizes vertical depth over linguistic breadth, making it possible to computationally analyze a wide spectrum of historical periods and confessional traditions within each language.

This GitHub mirror ships the corpus texts and metadata tables. The
companion HuggingFace dataset
([`mrapacz/targum-corpus`](https://huggingface.co/datasets/mrapacz/targum-corpus))
adds **pre-computed embeddings** (~120 GB) and **pairwise similarity
scores** (~5 GB).

## Corpus Scale

| Language | Code | Total | Unique | Public subset |
|---|---|---:|---:|---:|
| English | `eng` | 390 | 194 | 191 |
| French | `fra` | 78 | 41 | 43 |
| Spanish | `spa` | 102 | 53 | 29 |
| Polish | `pol` | 48 | 29 | 25 |
| Italian | `ita` | 33 | 17 | 13 |
| **Total** | | **651** | **334** | **301** |

"Total" is the number of translation instances collected across all 13 source libraries (the same translation may appear on multiple sites). "Unique" is the number of distinct translation editions after deduplication. "Public subset" is the number of instances distributed in this repository under public domain (235) or open licenses (66).

## Structure

```
corpus/
  {site}/
    {iso}/
      {id}.jsonl     # one verse per line
works.tsv            # one row per translation work
editions.tsv         # one row per edition, with copyright + provenance
instances.tsv        # one row per per-site instance, FK to editions
book_coverage.tsv    # which books each instance covers
manifest.json        # summary statistics
web/
  index.tsv           # denormalized viewer metadata
  landing.json        # precomputed landing-page data
  passages/
    {iso}/{BOOK}/{chapter}.jsonl.gz  # compact Compare passage shards
    manifest.json     # shard schema, provenance, and hashes

# Same tables are also available as .json (lists preserved as arrays).
```

Each JSONL file contains one verse per line:

```json
{"book": "MAT", "chapter": 1, "verse": "1", "text": "The book of the generation of Jesus Christ..."}
{"book": "MAT", "chapter": 1, "verse": "2", "text": "Abraham begat Isaac..."}
```

`book` uses USFM 3-letter New Testament codes: `MAT MRK LUK JHN ACT ROM 1CO 2CO GAL EPH PHP COL 1TH 2TH 1TI 2TI TIT PHM HEB JAS 1PE 2PE 1JN 2JN 3JN JUD REV`.

The derived `web/passages/` files group all available sources for one language
and chapter. They power the online Compare view without downloading a complete
New Testament file for every selected translation.

## Embeddings

Pre-computed text embeddings are available for all translations, produced by several encoder models at chapter and/or verse granularity:

| Model | Granularity | Files | Size |
|---|---|---:|---:|
| `Qwen/Qwen3-Embedding-0.6B` | chapter | 656 | ~269.6 MB |
| `Qwen/Qwen3-Embedding-0.6B` | verse | 5 | ~61.4 MB |
| `Qwen/Qwen3-Embedding-8B` | chapter | 656 | ~2.4 GB |

Embeddings are stored as Hive-partitioned Parquet files under `embeddings/` on [HuggingFace](https://huggingface.co/datasets/mrapacz/targum-corpus):

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
    "translation=engwebp/granularity=chapter/data.parquet"
)
```

## Similarity

The HuggingFace dataset also ships pairwise similarity scores between
translations under `similarity/`:

- **`similarity/lexical/jaccard/chapter/{iso}.parquet`** -- chapter-level Jaccard token-overlap.
- **`similarity/lexical/levenshtein/chapter/{iso}.parquet`** -- chapter-level Levenshtein-based scores.
- **`similarity/semantic/cosine/{model}/chapter/{iso}.parquet`** -- chapter-level cosine similarity between embeddings (currently `QwenXxXQwen3-Embedding-8B`); a `cross_language.parquet` adds pairs that span language boundaries.

One parquet file per language at chapter granularity. Total payload
is roughly 5 GB (lexical ~3 GB, semantic/cosine ~2 GB). See
[`docs/targum-corpus/similarity.md`](https://github.com/mrapacz/targum/blob/master/docs/targum-corpus/similarity.md)
for the exact definitions and reproduction steps.

## Metadata

Each translation has three layers of metadata:

- **`works.tsv`** — one row per translation work (`work_id`, display name, abbreviation, language, tradition, reference URLs).
- **`editions.tsv`** — one row per distinct edition (FK `work_id`), with curated `copyright_status` / `copyright_category` / `copyright_statement` / `copyright_statement_source_url` / `publisher` / `publication_year`.
- **`instances.tsv`** — one row per per-site instance (FK `edition_id`, `work_id`), with `site`, `iso`, `instance_id`, counts (`num_books`, `num_chapters`, `num_verses`, `num_words`) and the per-site declared copyright.

This three-layer structure lets researchers define "uniqueness" however they need: by work (translation family), by edition (specific revision), or by per-site instance.

## Quickstart

```python
import json
from pathlib import Path

corpus = Path("corpus")
# Load one translation
verses = [json.loads(line) for line in (corpus / "ebible.org/eng/engwebp.jsonl").read_text().splitlines()]
print(f"{len(verses)} verses loaded")

# Load all English translations
translations = {}
for path in sorted(corpus.glob("*/eng/*.jsonl")):
    translations[path.stem] = [json.loads(line) for line in path.read_text().splitlines()]
print(f"{len(translations)} English translations")
```

Or via HuggingFace datasets (note: path prefix is `corpora/` on HF):

```python
from datasets import load_dataset

ds = load_dataset("mrapacz/targum-corpus", data_files="corpora/ebible.org/eng/engwebp.jsonl", split="train")
```

## Source Data

Translations were collected from 13 libraries: bible.audio, bible.com, bible.is, biblegateway.com, biblehub.com, bibles.org, biblestudytools.com, bibliepolskie.pl, crossbible.com, ebible.org, jw.org, laparola.net, obohu.cz.

Only public domain and open-license translations are included in this release. The remaining 350 copyrighted translations are available to researchers upon reasonable request for noncommercial research purposes.

## Citation

If you use this corpus, please cite our LREC 2026 paper:

> Rapacz, M., & Smywiński-Pohl, A. (2026). Targum — a Multilingual New Testament Translation Corpus. In *Proceedings of the Fifteenth Language Resources and Evaluation Conference (LREC 2026)* (pp. 7092–7105). European Language Resources Association (ELRA). https://doi.org/10.63317/2yiotxcyovir

```bibtex
@inproceedings{rapacz-etal-2026-targum,
  title     = {Targum -- a Multilingual New Testament Translation Corpus},
  author    = {Rapacz, Maciej and Smywi{\'n}ski-Pohl, Aleksander},
  booktitle = {Proceedings of the Fifteenth Language Resources and Evaluation Conference (LREC 2026)},
  month     = {May},
  year      = {2026},
  pages     = {7092--7105},
  address   = {Palma, Mallorca, Spain},
  publisher = {European Language Resources Association (ELRA)},
  editor    = {Piperidis, Stelios and Bel, N{\'u}ria and van den Heuvel, Henk and Ide, Nancy and Krek, Simon and Toral, Antonio},
  doi       = {10.63317/2yiotxcyovir},
  url       = {https://lrec.elra.info/lrec2026-main-564}
}
```

Open-access PDF: <http://www.lrec-conf.org/proceedings/lrec2026/pdf/2026.lrec2026-1.564.pdf>.

## License

Corpus metadata and derived annotations are released under [CC-BY 4.0](LICENSE).
Individual translations retain their original licenses as recorded in `editions.tsv` (`copyright_statement` column).
