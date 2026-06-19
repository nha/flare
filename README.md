# Flare

**Turn any document into clean Markdown — locally, in one command.**

```bash
brew install nha/flare/flare
flare extract paper.pdf --json | jq -r '.items[].fts'
```

No uploads, no API keys, no per-page fees. Everything runs on your machine.

> `v0.0.1` · macOS Apple Silicon (arm64). This README covers `flare extract`.

---

## Install

```bash
brew install nha/flare/flare
flare --version          # flare-local 0.0.1 (java …)
```

## Extract a document to Markdown

Point `flare extract` at a file. By default it just extracts and prints — no index, no database, nothing stored. Output is structured (`--json` for one object, `--ndjson` for one item per line); each item's `.fts` field is the Markdown.

```bash
flare extract paper.pdf --json | jq -r '.items[].fts'
```

Real output, from the Transformer paper (`attention_is_all_you_need.pdf`):

```markdown
## Attention Is All You Need

| Ashish Vaswani* Google Brain | Noam Shazeer* Google Brain | Niki Parmar* Google Research |
| --- | --- | --- |
| Llion Jones* Google Research | Aidan N. Gomez* U. of Toronto | Lukasz Kaiser* Google Brain |

## Abstract

The dominant sequence transduction models are based on complex recurrent or
convolutional neural networks… We propose a new simple network architecture,
the Transformer, based solely on attention mechanisms…
```

Headings, the author block as a table, and the abstract — straight from the PDF. Scanned PDFs are OCR'd on-device, and tables come back as Markdown tables.

## File types

Same command, any of these:

```bash
flare extract report.pdf    --json | jq -r '.items[].fts'   # PDF (born-digital or scanned/OCR)
flare extract slides.pptx   --json | jq -r '.items[].fts'   # PowerPoint
flare extract notes.docx    --json | jq -r '.items[].fts'   # Word
flare extract budget.xlsx   --json | jq -r '.items[].fts'   # Excel
flare extract paper.odt     --json | jq -r '.items[].fts'   # OpenDocument (odt/ods/odp)
flare extract page.html     --json | jq -r '.items[].fts'   # HTML
flare extract scan.png      --json | jq -r '.items[].fts'   # Image (OCR)
flare extract inbox.mbox    --json | jq -r '.items[].fts'   # Email archive (mbox/eml)
flare extract meeting.m4a   --json | jq -r '.items[].fts'   # Audio/video → transcript
```

Supported: PDF (text + scanned/OCR), images, Office (`docx`/`pptx`/`xlsx`), OpenDocument (`odt`/`ods`/`odp`), HTML, Markdown/text, email (`mbox`/`eml`), and audio/video (on-device transcription).

## From a URL

```bash
flare extract https://example.com/post --json | jq -r '.items[].fts'
```

The page is fetched and extracted. Add `--full` to render JavaScript in a headless browser first.

## Output formats

```bash
# One JSON object: { mime, mode, items, count }
flare extract paper.pdf --json

# NDJSON — one item per line; composes with jq / wc / grep
flare extract big.pdf --ndjson | wc -l

# Just the Markdown text of every item
flare extract paper.pdf --json | jq -r '.items[].fts'
```

## Output fields

A `--json` response is `{ mime, mode, no_index, count, items }` — `mime` is the detected type, `mode` is `fast`/`full`, `count` the number of items. Each item:

| Field | What it is |
| --- | --- |
| `fts` | The extracted text (Markdown) — the full-text body; the main content. |
| `title` | A heading for the chunk (e.g. `Page 1: …`). |
| `page` | Page number, for paged formats (PDF). |
| `locator` | An **opaque** pointer to re-extract *this exact item*, e.g. `[{"kind":"pdf/page","page":1}]`. Pass it back via `--locator` to re-extract just this piece; copy it verbatim (the inner shape isn't a stable API). |
| `metadata` | Structured flags about the item, e.g. `{"table": true}`. |
| `dates` | Dates found in the text: `[{date-start, date-end, raw, kind}]` (ISO; `kind` is single/partial/range). |
| `embedding` | Semantic embedding vector (only with `--full`): base64 little-endian f32. Decode: base64 → LE f32 → vector. |
| `embedding-text` | Present only when the embedded text differs from `fts` (PDF sliding-window, email body vs headers) — the exact text fed to the embedder. |
| `color-hist` | Images / PDF figures: base64 288-d HSV colour histogram. |
| `clip-embedding` | Images / PDF figures: base64 512-d CLIP visual vector. |

Example item:

```json
{
  "title": "Page 1: Design and build accessible PDF tables",
  "page": 1,
  "locator": [{"kind": "pdf/page", "page": 1}],
  "metadata": {"table": true},
  "dates": [{"date-start": "2012-03-25", "date-end": "2012-03-25", "raw": "2012-03-25", "kind": "single"}],
  "fts": "PDF · Design and build accessible PDF tables …",
  "embedding": "T9cDPr1wjj8EQVPA…"
}
```

## Find lines inside a document (grep, but over parsed content)

Pass a pattern to keep only items whose text matches it — like `grep`, but over extracted Markdown (PDF/Office/email/…) rather than raw bytes:

```bash
flare extract statement.pdf "invoice number"
flare extract ~/Documents "hoxton"              # recurses a directory
```

## Options

| Flag | Effect |
| --- | --- |
| `--index` | Also store the file in the local index (launches the local app). |
| `--no-index` | Extract + print only — this is the **default** (kept for explicitness). |
| `--json` | One wrapped object `{ mime, mode, items, count }`. |
| `--ndjson` | Stream one JSON item per line (the default). |
| `--fast` | Skip OCR / transcription (text only, faster). |
| `--full` | Full pipeline incl. OCR / transcription (default); for URLs, render JS first. |
| `--ocr-language=<lang>` | Tesseract OCR language(s), e.g. `eng`, `fra`, `eng+fra`. |

Run `flare extract --help` for the complete list.

## Notes

- macOS Apple Silicon (arm64); early release (`v0.0.1`) — things may change.
- OCR uses Tesseract; quality on poor scans varies.

## License

[Elastic License 2.0](LICENSE.txt) — source-available: you can use, modify, and self-host Flare; you may not provide it to others as a managed service or remove its license-key functionality.
