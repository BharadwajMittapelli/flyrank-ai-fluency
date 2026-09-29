# 📚 The Polite Scraper

A polite, resilient web scraping pipeline built for the FlyRank Internship (Backend Track · Week 5 · Assignment A9). It collects book records from a sandbox environment, normalizes messy HTML into clean data, validates against a Pydantic schema, handles broken pages gracefully, and generates transparent execution reports.

---

## 🎯 Target Classification

- **Target Site**: Books to Scrape (`https://books.toscrape.com`)
- **Site Classification / Why**: Books to Scrape is an explicitly designed sandbox created specifically for testing and learning web scraping techniques ("a site built so people can practise scraping on it").
- **robots.txt Check**: A live request was sent to `https://books.toscrape.com/robots.txt` → **HTTP 404** (no robots file found). Permission is established through the site's documented role as a public scraping sandbox.
- **Collection Scope**: Strictly bounded to the first **3 catalogue pages**, discovering exactly **60 unique book URLs**.
- **Appropriateness**: Volume is non-disruptive, rate-limited with mandatory 500ms pauses between live requests, and confined to an open educational sandbox.

> **Ethical Commitment**:  
> *"I will not reuse this code on another site without checking its rules and terms first."*

---

## 📦 Data Schema

Each record in `output/books.json` conforms to the `BookRecord` Pydantic model:

| Field | Type | Description |
|:---|:---:|:---|
| `title` | `str` | Book title |
| `product_url` | `str` | Absolute canonical URL |
| `price_text` | `str` | Original raw price string (e.g. `"£51.77"`) |
| `price_gbp` | `float` | Normalized decimal price (e.g. `51.77`) |
| `availability_text` | `str` | Inventory status (e.g. `"In stock (22 available)"`) |
| `rating_text` | `str \| null` | Word rating: `"One"` to `"Five"` |
| `description` | `str \| null` | Book summary or `null` if missing |
| `source_page` | `str` | Provenance catalogue URL |
| `fetched_at` | `str` | UTC ISO 8601 timestamp of fetch |

---

## 📂 Project Structure

```
the-polite-scraper/
├── .gitignore           # Ignores cache/ and virtual environments
├── requirements.txt     # Pinned dependencies (requests, beautifulsoup4, pydantic)
├── README.md
├── src/
│   ├── fetcher.py       # Polite fetcher: User-Agent, local caching, 500ms pauses
│   ├── crawler.py       # Catalogue page link discovery across N pages
│   ├── extractor.py     # BS4 field extraction from book detail pages
│   ├── normalizer.py    # Price normalization (£51.77 → 51.77) and whitespace clean
│   ├── schema.py        # Pydantic BookRecord validation schema
│   └── main.py          # Pipeline orchestrator: crawl → fetch → extract → normalize → validate → report
└── output/
    ├── books.json        # 60 validated, clean records
    ├── errors.json       # Records that failed parse/validate (logged without crashing)
    └── report.json       # Run-level metadata: timings, counts, file paths
```

> **Note**: `cache/` is git-ignored. Cached HTML files are stored locally during a run to prevent redundant server requests.

---

## ⚙️ Prerequisites

- Python 3.10+

---

## 🚀 Installation & Running

### 1. Clone the repository
```bash
git clone https://github.com/BharadwajMittapelli/the-polite-scraper.git
cd the-polite-scraper
```

### 2. Create and activate virtual environment
```bash
# Windows
python -m venv .venv
.venv\Scripts\activate

# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Run the scraper
```bash
python src/main.py
```

On the **first run**, pages are fetched live from `books.toscrape.com` with **500ms pauses** between requests.  
On **subsequent runs**, all pages load from local `cache/` — instant and zero network load.

---

## 📤 Outputs

| File | Description |
|:---|:---|
| `output/books.json` | 60 validated clean records (deduplicated by `product_url`) |
| `output/errors.json` | Records that failed with reason (non-crashing) |
| `output/report.json` | Run metadata: timing, counts, discovery stats |

### Example Output Record
```json
{
  "title": "A Light in the Attic",
  "product_url": "https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html",
  "price_text": "£51.77",
  "price_gbp": 51.77,
  "availability_text": "In stock (22 available)",
  "rating_text": "Three",
  "description": "It's hard to imagine a world without A Light in the Attic...",
  "source_page": "https://books.toscrape.com/catalogue/page-1.html",
  "fetched_at": "2026-09-25T07:27:18.496681+00:00"
}
```

---

## 🐌 Politeness Features

| Feature | Implementation |
|:---|:---|
| **User-Agent** | `FlyRankInternship-A9/1.0 (+https://github.com/BharadwajMittapelli/the-polite-scraper)` |
| **Rate Limiting** | 500ms pause between every live network request |
| **Local Caching** | Pages cached as HTML files; never re-fetched during same session |
| **HTTP Verification** | Non-200 responses raise `RuntimeError`, never silently ignored |
| **Bounded Scope** | Strictly limited to 3 catalogue pages and 60 book URLs |

---

## 🛡️ Error Resilience

The pipeline uses **two-tier error isolation**:

1. **Fetch errors** (`RuntimeError` on HTTP failure) — logged to `errors.json`, pipeline continues.
2. **Parse/validate errors** (missing DOM elements, Pydantic `ValidationError`) — logged to `errors.json`, pipeline continues.

**A broken page never crashes the full run.** The error is captured with its URL, stage name, and reason message, and the scraper moves on.

### Broken Page Survival Test
```
SIMULATED BROKEN PAGE: https://books.toscrape.com/catalogue/broken-page_0/index.html
PARSE/VALIDATE ERROR: broken-page_0 — Target product area div.product_main not found
Pipeline completed: records_validated=60, records_errored=1
```

---

## 📊 Example Run Report

```json
{
  "run_at": "2026-09-25T07:37:21.883173+00:00",
  "finished_at": "2026-09-25T07:37:22.405911+00:00",
  "duration_seconds": 0.52,
  "catalogue_pages_crawled": 3,
  "book_urls_discovered": 60,
  "book_urls_unique": 60,
  "records_validated": 60,
  "records_errored": 1,
  "outputs": {
    "books_json": "output/books.json",
    "errors_json": "output/errors.json"
  }
}
```

---

## 🔀 Git Commit History

```
Stage 0: classify scraping target
Stage 1: fetch and cache HTML
Stage 2: discover three catalogue pages
Stage 3: extract book details
Stage 4: clean, validate, and store 60 book records
Stage 5: error survival and run report
Stage 6: publish to GitHub and write README
```
