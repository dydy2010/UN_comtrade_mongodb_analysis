# UN Comtrade MongoDB Analysis

This NoSQL project studies **Switzerland's exports using UN Comtrade**.
`UN_commodity.ipynb` keeps the validated World-by-chapter baseline for 2022-2025.
The working notebook, `UN_Comtrade_V2.ipynb`, studies 30 selected partner areas during 2023-2025.
Saved outputs confirm discovery, 90 HS4 batches, and 72,459 verified raw documents.
They also confirm 72,369 verified standard observations in `swiss_trade_v2.trade_clean`.

The current [project plan](PROJECT_PLAN.md) retains completed phases and sets the remaining submission checks.
V2 contains three executed analyses: destination shares, 2025 top-ten HS4 headings, and top-five destination HS2 composition.
The notebook now includes conclusions, learnings, sources, and the implemented model's UML image.
Final PDF review, controlled reproducibility checks, and packaging remain open.
The 4,235-observation top-50 subset remains an optional drill-down, not the source for complete chapter sums.

The remaining work uses only existing HS4 observations and their saved reference metadata. `partner_totals_raw` contains all-commodity totals, not HS2 records, and remains historical discovery evidence. Derive HS2 summaries from HS4 parent codes. No partner-total clean collection or new extraction is planned. Future market shares refer to the selected standard-HS4 dataset, not all Swiss exports.

## V1 baseline scope

- Reporter: Switzerland, Comtrade code **757** (not ISO numeric code 756).
- Flow: exports of goods; frequency: annual; partner: World.
- Products: HS chapters 01–97 excluding reserved chapter 77, requested with `cmdCode="AG2"`.
- Classification: original HS requested, with returned edition required to be **H6 (HS 2022)** for all years.
- Expected data: **96 chapters per year × 4 years = 384 records**, based on public API checks. Partial-year 2026 monthly data is excluded from the annual comparison.
- Measures: current-US-dollar trade values, plus raw weights, quantities, and quality flags where available.

The analysis will identify leading chapters, changes in chapter shares and export values, and top-five concentration. Shares refer to the sum of included chapters unless separately reconciled with an official total. We do not infer inflation-adjusted growth, company profits, or physical-volume changes from dollar values alone.

All analysis will use MongoDB pipelines after loading. The intended clean model embeds related reporter, partner, commodity, measure, and quality fields within each year/chapter observation; a separate category tree is not needed for this scope.

## V2 partner discovery (completed)

V2 requests annual Swiss export totals using `partnerCode=None` and `cmdCode="TOTAL"`. Python omits the partner filter, so the response includes reported partners and World. The notebook ranks the top 30 non-World partner areas by cumulative 2023–2025 value. Code `490`, `Other Asia, nes`, remains eligible under the partner-area scope. HS4 extraction then retrieves headings for those partners, and MongoDB ranks 50 headings across the selected partners and years. V1 remains unchanged.

V2 uses the same API-key setup and Python environment described below. Its discovery snapshot and CSV use V2-specific filenames so they are not confused with V1 data. Do not reuse the V1 snapshot for V2.

## Run the notebook

From this repository's folder:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m notebook
```

Alternatively, select an existing Python environment with these dependencies in your editor. Open `UN_commodity.ipynb` and use that environment as its kernel. The kernel's working directory should be this repository folder.

`UN_Comtrade_V2.ipynb` uses the same environment and `.env` API key. Discovery and HS4 extraction have separate source settings and snapshots. Review the notebook's connection and write cells before running everything; the user may have enabled earlier Atlas operations.

### Option A: fresh API extraction

1. Register or sign in to UN Comtrade using the [free account guide](https://uncomtrade.org/docs/how-to-create-an-account/).
2. Open [My API Portal](https://comtradeplus.un.org/MyComtrade/APIPortal), generate a key if needed, and copy either the primary or secondary key. Free Basic Individual access is sufficient for this scope.
3. Set `DATA_SOURCE = "api"`, copy `.env.example` to `.env`, and fill in `UN_COMTRADE_API_KEY=` with your key. Restart the kernel and run from the top.

Reference: UN Comtrade's [official example notebook](https://github.com/uncomtrade/comtradeapicall/blob/main/tests/example%20calling%20functions%20-%20notebook.ipynb) demonstrates its `comtradeapicall` helper package. This project uses `requests` directly to follow the course approach; that helper package is not required.

The local `.env` format is:

```dotenv
UN_COMTRADE_API_KEY=
```

Fill in your primary or secondary Comtrade key locally. Do not use both. The notebook loads this file, or prompts securely when no key is available. Existing environment values take precedence; restart the kernel after changing the file.

Never commit or include `.env` in the submission ZIP. Git ignore rules do not protect a file that was already tracked.

Run the notebook from the top and save the outputs. A complete extraction should contain 384 unique year/chapter records with matching query dimensions, H6 classification, and a complete extraction log. Missing chapters stop validation rather than being silently filled with zero. HTTP 401/403 requires checking account/key access; HTTP 429 indicates a rate limit. Do not publish credentials or request headers when asking for help.

After validation, a saving cell creates a dated JSON snapshot under `data/`. It contains raw records, query settings, and download timestamps, but no headers or credentials. Existing snapshots are not overwritten. Saving local JSON does not replace loading and transforming data inside MongoDB.

### Option B: reproduce from supplied data

No Comtrade registration or key is needed for snapshot mode:

1. Obtain the snapshot supplied with the project and keep it in `data/`.
2. Change `DATA_SOURCE` to `"snapshot"` and set `SNAPSHOT_PATH` to that exact dated filename.
3. Restart the kernel and run the notebook from the top.

The notebook skips key loading and API requests, reads the raw records and extraction log, checks the query scope, and applies the same validation. Earlier US pilot or 2022–2024 Swiss snapshots do not match the current scope and cannot be reused without explicitly changing the settings. It does not overwrite the supplied snapshot or silently fall back to the API. Future MongoDB steps will still need an appropriate database connection.

The first snapshot does not exist until the API extraction, validation, and saving cells have run. The path printed by the saving cell is the one to use for reproduction. JSON is suitable here because the raw API data uses JSON types; future MongoDB BSON exports may need Extended JSON to preserve dates and ObjectIds.

### Submission and data distribution

The assignment text supplied for this project requires the PDF, notebook, and images in a ZIP; it does not explicitly forbid dataset files or confirm whether additional files are accepted. Check the course's upload guide or ask the professor before assuming that a snapshot is accepted as an extra file.

UN Comtrade has specific [use and re-dissemination terms](https://uncomtrade.org/docs/re-dissemination-of-data/), not an unrestricted open-data license. Check the applicable exceptions and attribution requirements for the intended distribution. A small assessment snapshot and a public raw-data repository are different uses. Do not publish a larger raw dataset without checking the terms.

If accepted, include the selected dated snapshot, `.env.example`, `requirements.txt`, and these instructions alongside the required files. Exclude `.env` and unused snapshots. Preserve the exact snapshot used for the submitted results and identify its filename in the notebook.

## Check validation without API access

```bash
python3 -m unittest -q test_pilot_validation test_india_exploration test_comtrade_v2 test_comtrade_hs4 test_comtrade_quality test_comtrade_transform test_comtrade_save test_comtrade_partners test_comtrade_products
```

These tests use synthetic records and do not read `.env`, make API requests, or write to MongoDB.

The current suite passes 146 offline tests.
The tests use mocks; they do not replace the notebook's saved live MongoDB evidence.
V2 contains completed quality checks, transformation previews, saved-collection verification, and analysis outputs.
The current notebook enables connection, quality checking, verification, and the three analyses.
It keeps `HS4_DATA_SOURCE = "off"` and `SAVE_TRADE_CLEAN = False`.
Review all earlier extraction and write flags before executing cells.
Do not run the notebook automatically to refresh documentation or export a PDF.

## Check the Atlas connection

1. Install the updated project requirements in the notebook's Python environment.
2. Add `MONGODB_URI` to your local `.env`, preserving the Comtrade key.
3. Run the V2 imports cell, then find **Database setup: connect to Atlas** after the extraction log.
4. Set `CONNECT_TO_MONGODB = True` and run the settings and connection cells.

The code follows the course's `pymongo.MongoClient(CNX_STR)` pattern and verifies connectivity with `client.admin.command("ping")`. The connection cell does not write data. You do not need to rerun the API extraction. Set the connection flag to `False` to skip configuration loading and Atlas access. Existing environment values take precedence over `.env`.

Never print the URI or include `.env` in a commit or submission.

## Historical HS4 extraction procedure

The current project already has the required HS4 observations and frozen reference.
The steps below document the earlier extraction; they do not describe the remaining work.
Do not request new data for the final report.

The partner totals must already exist, and the partner-ranking and selection cells must have produced 30 codes.

1. Rerun the updated download-function cell. This defines the shared request helper without downloading data.
2. In **Extract HS4 detail for the selected partners**, set `HS4_DATA_SOURCE = "api"`.
3. Keep `LOAD_HS4_TO_MONGODB` and `RANK_HS4_IN_MONGODB` set to `False`.
4. Run the HS4 settings, validation-functions, reference-and-coverage-functions, and extraction cells in order.
5. Save the snapshot, then inspect the preview, batch counts, nonstandard codes, and coverage audit.
6. Copy the printed dated path into `HS4_SNAPSHOT_PATH` and switch to `HS4_DATA_SOURCE = "snapshot"` for later runs.
7. Run the MongoDB helper cell, enable `LOAD_HS4_TO_MONGODB`, and run only the HS4 load cell.
8. After verifying the load, enable `RANK_HS4_IN_MONGODB` and run the read-only ranking cell.

The first download makes 90 data requests before retries, plus one public H6 reference download. It requests one partner per year, with at least five seconds between data batches. Check your remaining quota first. HS4 API mode loads its own key even when discovery uses snapshot mode. New HS4 settings default to `off` with both database flags disabled.

HS4 snapshot schema version 2 preserves the selected partners, raw H6 reference, and all-commodity control totals alongside records and logs. Snapshot mode needs no reference download or Atlas access in the extraction cells. It rejects incompatible scope, classification, duplicate keys, invalid values, and inconsistent logs. Empty batches remain visible; missing observations do not become artificial zeroes.

The coverage audit uses familiar Pandas grouping and joins to compare source-row sums and standard-heading sums with frozen partner totals. It reports differences without rescaling values. Missing rows and zero denominators retain undefined coverage. Investigate differences before interpreting rankings.

The raw loader preserves source fields and stable IDs while using standard PyMongo `ReplaceOne` and `bulk_write()` batches of at most 1,000. This extends the course's replacement/upsert pattern to reduce Atlas round trips. It refuses stale in-scope observations before writing and never deletes them automatically. Ranking uses MongoDB to select 50 standard H6 headings overall; nonstandard rows remain raw but do not enter this ranking.

The completed clean model retains all standard HS4 observations, not just the top 50.
MongoDB derives `product.chapter.hsCode` from each heading prefix.
The saved H6 reference supplies chapter labels.
Keep the verified `trade_clean` destination and its completed checks unchanged.

## Optional India exploration

This side project is kept in the standalone `India_HS2710_exploration.ipynb` notebook and is separate from the Swiss analysis. Run its exploration cell to retrieve India's calendar-year 2025 imports of HS 2710 from all available partners through the no-key public API. It exports an Excel workbook and a top-15 value-ranking chart to `data/india_exploration/`. HS 2710 excludes crude oil; partner values are not physical quantities.

The workbook has readable supplier rankings, a separate World total, all raw fields, reconciliation checks, query details, and an embedded chart. Partner countries/areas may include unspecified categories. This notebook does not change the Swiss data or settings.

## Render the PDF

Install Quarto and a working TeX distribution separately if not already available. Quarto's standard PDF workflow can use TinyTeX.

After the selected project notebook has its final analysis, diagrams, and saved outputs:

```bash
quarto render "UN_commodity.ipynb" --to pdf --no-execute
```

For the V2 final report, replace the filename with `UN_Comtrade_V2.ipynb`. Rendering uses saved outputs and does not execute the API cells. Regenerate the PDF after notebook changes; an existing PDF may be an older preview. Inspect the final tables, images, code layout, and page breaks before submission.