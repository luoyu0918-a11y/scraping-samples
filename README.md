```
scraping-samples/
├── README.md
├── requirements.txt
├── .gitignore
├── books-toscrape/
│   ├── books.csv
│   ├── scrape.py
│   └── README.md
└── sra-kantoorvinder/
    ├── sample_50.csv
    ├── scrape.py
    ├── README.md
    └── drift_check.md
```

# scraping-samples

Two extracts, both produced with the scripts in this repo. One is a paginated
HTML catalogue, one is a JSON-backed directory behind a JS-rendered page — the
two situations most scraping jobs actually fall into.

| Project | Records | Shape | Method |
|---|---|---|---|
| [`books-toscrape/`](books-toscrape) | 1,000 | 50 paginated HTML pages, 20 products each | `requests` + `BeautifulSoup`, next-page loop with three termination guards |
| [`sra-kantoorvinder/`](sra-kantoorvinder) | 1,030 | Single JSON endpoint behind a JS shell | endpoint discovery → one request → field mapping, normalisation, dedupe |

## What's worth looking at

**Pagination that terminates.** `books-toscrape/scrape.py` stops on three
independent conditions: empty result page, duplicate page content (md5), and a
product URL already seen. Any one of those alone will eventually loop forever on
a site that starts repeating results; all three together won't.

**Finding the endpoint instead of rendering the page.** `sra-kantoorvinder/scrape.py`
never opens a browser. The directory on the page is loaded by a single XHR call,
so the script calls that endpoint directly — one request instead of ~1,030 page
renders, and no headless-browser maintenance.

**Normalisation is part of the extract, not an afterthought.** Postcodes
normalised to `1234 AB`, duplicate rows collapsed on a composite key
(firm + postcode + house number), Dutch accents left intact, and town names kept
in the source's official form including province suffixes (`KESSEL LB`).

**Every row is traceable.** Each record carries the URL it came from and the
timestamp it was captured. No row is inferred, filled or guessed.

## Data handling note

The SRA sample contains business contact details, so email addresses and phone
numbers are partially masked in `sample_50.csv`. The script writes the full
values; the published sample is 50 rows of an otherwise 1,030-row extract.

## Running either one

```bash
pip install -r requirements.txt
python books-toscrape/scrape.py --out books.csv
python sra-kantoorvinder/scrape.py --out sra_offices.csv
```

Both scripts take `--out`, both write UTF-8 with BOM so Excel opens them without
mojibake, and both print a per-page or per-record progress line so you can see
where a run stopped.
