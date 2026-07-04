---
language:
- en
- fr
- es
- pl
- it
license: cc-by-4.0
pretty_name: Targum Corpus
multilinguality:
- multilingual
tags:
- bible
- bible-translation
- translation
- nlp
- corpus-linguistics
- parallel-corpus
- digital-humanities
- religion
size_categories:
- 1M<n<10M
task_categories:
- text-generation
- translation
configs:
- config_name: corpora
  data_files: "corpora/**/*.jsonl"
  default: true
- config_name: works
  data_files: "works.tsv"
- config_name: editions
  data_files: "editions.tsv"
- config_name: instances
  data_files: "instances.tsv"
- config_name: book_coverage
  data_files: "book_coverage.tsv"
---

# Targum -- A Multilingual New Testament Translation Corpus

**Links:** 📄 [LREC 2026 paper](https://aclanthology.org/2026.lrec-main.564/) · 🗂️ [GitHub mirror](https://github.com/mrapacz/targum-corpus) · 🌐 [Online viewer](https://targum.mrapacz.com/)

**Targum** is a multilingual New Testament translation corpus with unprecedented depth in five European languages: English, French, Italian, Polish, and Spanish. It contains **651** translations (**334** unique) collected from **13** source libraries and spanning **1525–2025**.

This dataset contains the **public release subset**: **302** translations distributed under public domain or open licenses. The remaining 349 copyrighted translations are not distributed but are available to researchers upon request.

Named after the ancient Aramaic translations of the Hebrew Bible (תרגום, "translation"), the corpus prioritizes vertical depth over linguistic breadth, making it possible to computationally analyze a wide spectrum of historical periods and confessional traditions within each language.

The repository ships three families of artefacts side by side:

- **Corpus texts** -- verse-level JSONL under `corpora/`.
- **Embeddings** -- pre-computed encoder vectors under `embeddings/` (~120 GB across six (model, granularity) combinations).
- **Similarity** -- pairwise lexical and semantic similarity scores under `similarity/` (~5 GB).

The metadata tables (`works`, `editions`, `instances`, `book_coverage`) are exposed as HuggingFace dataset configs and can be loaded directly with `load_dataset`.

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
works.tsv               # one row per translation work (a work groups all editions of one underlying translation)
editions.tsv            # one row per edition, with copyright + provenance
instances.tsv           # one row per per-site instance, FK to editions
book_coverage.tsv       # which books each instance covers
manifest.json           # summary statistics

# Same tables are also available as .json (lists preserved as arrays).
```

Each JSONL file:

```json
{"book": "MAT", "chapter": 1, "verse": "1", "text": "The book of the generation of Jesus Christ..."}
{"book": "MAT", "chapter": 1, "verse": "2", "text": "Abraham begat Isaac..."}
```

`book` uses USFM 3-letter New Testament codes: `MAT MRK LUK JHN ACT ROM 1CO 2CO GAL EPH PHP COL 1TH 2TH 1TI 2TI TIT PHM HEB JAS 1PE 2PE 1JN 2JN 3JN JUD REV`.

## Embeddings

Pre-computed text embeddings are available for all translations, produced by several encoder models at chapter and/or verse granularity:

| Model | Granularity | Files | Size |
|---|---|---:|---:|
| `Qwen/Qwen3-Embedding-0.6B` | verse | 656 | ~7.7 GB |
| `Qwen/Qwen3-Embedding-8B` | chapter | 656 | ~1007.1 MB |
| `Qwen/Qwen3-Embedding-8B` | verse | 656 | ~74.5 GB |
| `nvidia/llama-embed-nemotron-8b` | chapter | 656 | ~1007.5 MB |
| `sentence-transformers/LaBSE` | chapter | 656 | ~1.1 GB |
| `sentence-transformers/LaBSE` | verse | 656 | ~22.8 GB |

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
import pyarrow.parquet as pq
from huggingface_hub import hf_hub_download

p = hf_hub_download(
    repo_id="mrapacz/targum-corpus",
    repo_type="dataset",
    filename="embeddings/QwenXxXQwen3-Embedding-0.6B/language=eng/"
             "site=ebible.org/translation=engwebp/granularity=chapter/data.parquet",
)
df = pq.ParquetFile(p).read().to_pandas()
```

## Similarity

Pre-computed pairwise similarity scores between translations are
distributed alongside the corpus under `similarity/`:

- **`similarity/lexical/jaccard/chapter/{iso}.parquet`** -- chapter-level Jaccard token-overlap scores.
- **`similarity/lexical/levenshtein/chapter/{iso}.parquet`** -- chapter-level Levenshtein-based scores.
- **`similarity/semantic/cosine/{model}/chapter/{iso}.parquet`** -- chapter-level cosine similarity between embeddings (currently `QwenXxXQwen3-Embedding-8B`). A `cross_language.parquet` adds pairs that span language boundaries.

One parquet file per language at chapter granularity. Total payload
is roughly 5 GB (lexical ~3 GB, semantic/cosine ~2 GB).

```python
import pandas as pd

sem = pd.read_parquet(
    "hf://datasets/mrapacz/targum-corpus/similarity/semantic/cosine/"
    "QwenXxXQwen3-Embedding-8B/chapter/ita.parquet"
)
print(sem.head())
```

## Metadata

Each translation has three layers of metadata:

- **`works.tsv`** — one row per translation work (`work_id`, display name, abbreviation, language, tradition, reference URLs). A work groups together all editions of the same underlying translation.
- **`editions.tsv`** — one row per distinct edition (FK `work_id`). Carries the curated copyright status / category / statement / source URL / publisher / publication year — these are invariant across the sites that host the edition.
- **`instances.tsv`** — one row per per-site instance of an edition (FK `edition_id` → `work_id`). Captures the per-site declared copyright (which may disagree with the curated truth — useful for spotting publisher disputes).

This three-layer structure lets researchers define "uniqueness" for their own needs: deduplicate by work (translation family), edition (specific revision), or keep every per-site instance.

## Usage

### Verse text

```python
from datasets import load_dataset

# Load a single translation
ds = load_dataset(
    "mrapacz/targum-corpus",
    data_files="corpora/ebible.org/eng/engwebp.jsonl",
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

### Metadata via configs

Each declared config maps to one of the TSV tables and is loadable
directly:

```python
from datasets import load_dataset

works       = load_dataset("mrapacz/targum-corpus", "works",          split="train")
editions    = load_dataset("mrapacz/targum-corpus", "editions",       split="train")
instances   = load_dataset("mrapacz/targum-corpus", "instances",      split="train")
book_cov    = load_dataset("mrapacz/targum-corpus", "book_coverage",  split="train")
```

Or as pandas / polars frames over the raw TSV / JSON files:

```python
import pandas as pd

editions  = pd.read_csv("hf://datasets/mrapacz/targum-corpus/editions.tsv",  sep="\t")
instances = pd.read_csv("hf://datasets/mrapacz/targum-corpus/instances.tsv", sep="\t")
```

### Selective access (recommended for embeddings / similarity)

With ~120 GB of embeddings + ~5 GB of similarity, you almost never
want a full clone. Pull only what you need:

```python
from huggingface_hub import hf_hub_download, snapshot_download

# One specific embedding file
path = hf_hub_download(
    repo_id="mrapacz/targum-corpus",
    repo_type="dataset",
    filename="embeddings/QwenXxXQwen3-Embedding-0.6B/language=eng/"
             "site=ebible.org/translation=engwebp/granularity=chapter/data.parquet",
)

# All English chapter embeddings for one model
local_dir = snapshot_download(
    repo_id="mrapacz/targum-corpus",
    repo_type="dataset",
    allow_patterns=[
        "embeddings/QwenXxXQwen3-Embedding-0.6B/language=eng/**/granularity=chapter/data.parquet",
    ],
)

# All metadata + the public corpus, no embeddings, no similarity
local_dir = snapshot_download(
    repo_id="mrapacz/targum-corpus",
    repo_type="dataset",
    allow_patterns=["*.tsv", "*.json", "corpora/**"],
)
```

The repository is backed by HuggingFace's [Xet](https://huggingface.co/docs/hub/storage-backends#xet)
storage. To pull only the slices you care about from the
shell, use the [`hf` CLI](https://huggingface.co/docs/huggingface_hub/main/en/guides/cli)'s
`--include` filters:

```bash
pip install -U "huggingface_hub[cli]"

# One specific embedding file
hf download mrapacz/targum-corpus --repo-type dataset \
  --include "embeddings/QwenXxXQwen3-Embedding-0.6B/language=eng/site=ebible.org/translation=engwebp/granularity=chapter/data.parquet" \
  --local-dir ./targum-corpus

# All English chapter embeddings for one model
hf download mrapacz/targum-corpus --repo-type dataset \
  --include "embeddings/QwenXxXQwen3-Embedding-0.6B/language=eng/**/granularity=chapter/data.parquet" \
  --local-dir ./targum-corpus

# Metadata + public corpus only (no embeddings, no similarity)
hf download mrapacz/targum-corpus --repo-type dataset \
  --include "*.tsv" --include "*.json" --include "corpora/**" \
  --local-dir ./targum-corpus
```

## Source Data

Translations were collected from 13 libraries: bible.audio, bible.com, bible.is, biblegateway.com, biblehub.com, bibles.org, biblestudytools.com, bibliepolskie.pl, crossbible.com, ebible.org, jw.org, laparola.net, obohu.cz.

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

Corpus metadata and derived annotations: [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Individual translations retain their original licenses as recorded in `editions.tsv` (`copyright_statement` column).
