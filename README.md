<p align="center"><a href="https://apify.com/DevilScrapes/uspto-patents-scraper"><img src="assets/banner.svg" alt="USPTO Patent Scraper — Python example" width="100%"></a></p>

# USPTO Patents Scraper in Python: US patent search results to pandas, JSON and CSV

<p align="center">
<a href="https://apify.com/DevilScrapes/uspto-patents-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/uspto-patents-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/uspto-patents-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

This repository is a short, tested Python client for the **[USPTO Patent Scraper](https://apify.com/DevilScrapes/uspto-patents-scraper)** on Apify. Write a query the way you would type it into a patent search box, for example `TI/(solid state battery)` or `AN/` followed by a company, and the script returns US patents and published applications as rows: number, title, abstract, inventors, assignees, CPC codes, kind code and the four key dates. The search runs on Apify against Google Patents' index of the USPTO corpus, so you need no patent-office account and no key to manage. Results are saved as `results.json` and `results.csv`, which is the format notebooks, pandas and spreadsheet-loving patent analysts all agree on.

> **Actor:** [USPTO Patent Scraper](https://apify.com/DevilScrapes/uspto-patents-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [uspto-patents-scraper-javascript](https://github.com/devilscrapes/uspto-patents-scraper-javascript)

## 🔥 What you get

- **US documents only**: every search is restricted to US grants and published applications, so a sweep on one company does not fill up with foreign family members of the same invention.
- **Search-box qualifiers that work**: `TI/` for titles, `AB/` for abstracts, `AN/` for assignee, `IN/` for inventor and `CPC/` for a classification, one qualifier per query, or plain words across title, abstract and claims.
- **Three orderings**: relevance, newest publication first or oldest publication first.
- **Grants told apart from applications**: `grant_date` is filled only for B1 and B2 kind codes, while `publication_date`, `filing_date` and `priority_date` arrive as ISO `YYYY-MM-DD` strings.
- **Lists stay lists**: inventors, assignees and CPC codes come back as arrays, never as one comma-joined string you have to split.
- **Stops where you told it to**: the `maxResults` cap (up to 2,000 per run) is enforced in the middle of a page, so you never pay for overshoot.
- **One failed document never sinks the run**: a patent page that errors is logged and skipped, it is not billed, and the rest of the result set still lands.
- **Every row links home**: `patent_url` points to the full document on Google Patents for claims, drawings and citations.

## 🎯 Use cases

### Competitor portfolio watch on a schedule

Run an `AN/` query for each competitor from cron every week with `sortBy` set to newest first, and keep the CSVs by date. A pandas anti-join on `patent_number` against last week's file lists only the new publications, which is the part an IP team actually needs to read.

### Technology landscape by CPC subclass

Pull a few hundred documents for a technical phrase, explode `cpc_classifications` and count documents per four-character subclass. A bar chart of those counts in a notebook shows where the activity sits and which neighbouring classes deserve a closer search.

### Filing-trend analysis for a research paper

Convert `priority_date` and `publication_date` with `pd.to_datetime`, resample by quarter and compare the curves for two technologies. Because every date is already normalised, the cleaning step is one line instead of an afternoon.

### Abstract corpus for text mining

Collect titles and abstracts for a class or a keyword and feed them to a topic model, an embedding job or a classifier. Each document keeps its number and URL, so any cluster you find can be traced back to the original patent.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/uspto-patents-scraper-python.git
cd uspto-patents-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [USPTO Patent Scraper](https://apify.com/DevilScrapes/uspto-patents-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Here is everything the integration really needs. The `apify-client` package launches the run, waits until it is done and iterates through the dataset for you.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/uspto-patents-scraper").call(run_input={'searchQuery': 'TI/(solid state battery)', 'maxResults': 10, 'sortBy': 'publication_date_desc', 'proxyConfiguration': {'useApifyProxy': False}})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

The script passes `input.json` to the Actor untouched. The included example searches patent titles for one phrase, sorts by newest publication and stops at ten documents, a run that costs a few cents. Swap in `AN/` plus a company name for an assignee sweep, `CPC/` plus a code for a classification sweep, and raise `maxResults` once the query returns what you expect.

```json
{
  "searchQuery": "TI/(solid state battery)",
  "maxResults": 10,
  "sortBy": "publication_date_desc",
  "proxyConfiguration": {
    "useApifyProxy": false
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `searchQuery` | string | yes | `"neural network training"` | — |
| `maxResults` | integer | no | `30` | — |
| `sortBy` | string | no | `"relevance"` | `relevance`, `publication_date_desc`, `publication_date_asc` |
| `proxyConfiguration` | object | no | `{"useApifyProxy": false}` | — |

## 📤 Output

Each row is one US document. The list columns (`inventors`, `assignees`, `cpc_classifications`) load into pandas as Python lists, ready for `explode`; date columns are ISO strings or `None`, and `grant_date` is `None` for anything that is not a granted B1 or B2 document. In current builds `applicants` and `ipc_classifications` are present but empty lists, so key your analysis on assignees and CPC codes.

| Field | Type | Always present |
|---|---|---|
| `patent_number` | string | yes |
| `publication_id` | string | no |
| `title` | string | yes |
| `abstract` | string or null | no |
| `inventors` | array | no |
| `assignees` | array | no |
| `applicants` | array | no |
| `cpc_classifications` | array | no |
| `ipc_classifications` | array | no |
| `publication_date` | string or null | no |
| `grant_date` | string or null | no |
| `filing_date` | string or null | no |
| `priority_date` | string or null | no |
| `kind_code` | string or null | no |
| `patent_url` | string | yes |
| `scraped_at` | string | no |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result emitted | $0.003 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/uspto-patents-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

This pandas example (`pip install pandas`) turns the raw rows into a quick landscape: the busiest CPC subclasses, the most frequent assignees and a monthly publication count, each saved as a CSV.

```python
import json

import pandas as pd

rows = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))
rows["published"] = pd.to_datetime(rows["publication_date"], errors="coerce")
rows["granted"] = rows["grant_date"].notna()

cpc = rows.explode("cpc_classifications").dropna(subset=["cpc_classifications"])
cpc["subclass"] = cpc["cpc_classifications"].str[:4]
subclasses = cpc.groupby("subclass")["patent_number"].nunique().sort_values(ascending=False)
subclasses.head(15).to_csv("cpc_subclasses.csv")

owners = rows.explode("assignees").dropna(subset=["assignees"])
top_owners = owners.groupby("assignees").agg(documents=("patent_number", "nunique"), grants=("granted", "sum"))
top_owners.sort_values("documents", ascending=False).head(15).to_csv("assignees.csv")

dated = rows.dropna(subset=["published"]).set_index("published")
monthly = dated.resample("MS")["patent_number"].count()
monthly.to_csv("publications_by_month.csv")
print(subclasses.head(10).to_string())
```

## ❓ FAQ

### Do I need a USPTO account or a patent API key?

No. An Apify API token is the only credential. The Actor reads Google Patents' public index of US documents, so there is no developer registration or quota approval on your side.

### Can I combine an assignee and a title condition in one query?

Not yet. Each query takes plain text or a single qualifier, and boolean operators between qualifiers are not supported. Run one query per condition, then intersect the results in pandas with a merge on `patent_number`.

### How current is the data?

The USPTO usually issues grants on Tuesdays and Google Patents tends to index new publications within about a week. Weekly cron runs fit that rhythm.

### Does it return claims text or patent families?

No. Full claims, drawings, inventor addresses and foreign family members are outside its scope, and so are EPO and WIPO documents. Follow `patent_url` for the full text, and use the Actor for the metadata you filter and count.

### Is this good enough for a freedom-to-operate opinion?

It is a research and screening tool. It speeds up the first pass, but a legal opinion still belongs with a registered patent attorney, and anything that must match the official record should be checked against the USPTO itself.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/uspto-patents-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [USPTO Patent Scraper on Apify Store](https://apify.com/DevilScrapes/uspto-patents-scraper)
- [JavaScript version of this example](https://github.com/devilscrapes/uspto-patents-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [USPTO Trademark Scraper](https://apify.com/DevilScrapes/uspto-trademark-scraper) — the same office's trademark records, searched by mark, owner or number.
- [EUIPO Trademark Search Scraper](https://apify.com/DevilScrapes/euipo-trademark-search-scraper) — European trademark filings for brand-clearance checks next to patent work.
- [arXiv Scraper — Search & Export Paper Metadata](https://apify.com/DevilScrapes/arxiv-papers-scraper) — research preprints, useful for prior-art and technology-trend context.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/uspto-patents-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with the USPTO or Google.
