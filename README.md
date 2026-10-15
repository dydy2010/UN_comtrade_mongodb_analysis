# UN Comtrade MongoDB Analysis

This NoSQL project studies **Switzerland's export structure from 2022 to 2025**. Extraction, validation, and snapshot saving are implemented; MongoDB loading, document modeling, and aggregation analysis are the next stages.

## Research scope

- Reporter: Switzerland, Comtrade code **757** (not ISO numeric code 756).
- Flow: exports of goods; frequency: annual; partner: World.
- Products: HS chapters 01–97 excluding reserved chapter 77, requested with `cmdCode="AG2"`.
- Classification: original HS requested, with returned edition required to be **H6 (HS 2022)** for all years.
- Expected data: **96 chapters per year × 4 years = 384 records**, based on public API checks. Partial-year 2026 monthly data is excluded from the annual comparison.
- Measures: current-US-dollar trade values, plus raw weights, quantities, and quality flags where available.

The analysis will identify leading chapters, changes in chapter shares and export values, and top-five concentration. Shares refer to the sum of included chapters unless separately reconciled with an official total. We do not infer inflation-adjusted growth, company profits, or physical-volume changes from dollar values alone.

All analysis will use MongoDB pipelines after loading. The intended clean model embeds related reporter, partner, commodity, measure, and quality fields within each year/chapter observation; a separate category tree is not needed for this scope.

## Run the notebook

From this repository's folder:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m notebook
```

Alternatively, select an existing Python environment with these dependencies in your editor. Open `UN_commodity.ipynb` and use that environment as its kernel. The kernel's working directory should be this repository folder.

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
python -m unittest -v test_pilot_validation test_india_exploration
```

These tests use synthetic records and do not read `.env`, make API requests, or write to MongoDB.

## Optional India exploration

The last notebook cell is separate from the Swiss project. Set `RUN_INDIA_EXPLORATION = True` to retrieve India's calendar-year 2025 imports of HS 2710 from all available partners through the no-key public API. It exports an Excel workbook and a top-15 value-ranking chart to `data/india_exploration/`. HS 2710 excludes crude oil; partner values are not physical quantities.

The workbook has readable supplier rankings, a separate World total, all raw fields, reconciliation checks, query details, and an embedded chart. Partner countries/areas may include unspecified categories. Set the switch back to `False` when returning to normal Swiss runs. The optional cell is excluded from Quarto output and does not change the Swiss data or settings.

## Render the PDF

Install Quarto and a working TeX distribution separately if not already available. Quarto's standard PDF workflow can use TinyTeX.

After running and saving the notebook:

```bash
quarto render "UN_commodity.ipynb" --to pdf --no-execute
```

Rendering uses saved outputs and does not execute the API cells. Regenerate the PDF after notebook changes; an existing PDF may be an older preview. Inspect the final tables, images, code layout, and page breaks before submission.