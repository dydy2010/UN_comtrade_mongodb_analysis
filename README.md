# UN Comtrade MongoDB Analysis

This repository currently contains an extraction pilot, not the completed NoSQL project. It requests US exports of HS chapter 85 to the world for 2022 and 2023. MongoDB loading and analysis will follow after the research scope is chosen.

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
2. Open [My API Portal](https://comtradeplus.un.org/MyComtrade/APIPortal), generate a key if needed, and copy either the primary or secondary key. Free Basic Individual access is sufficient for this pilot.
3. Set `DATA_SOURCE = "api"`, copy `.env.example` to `.env`, and fill in `UN_COMTRADE_API_KEY=` with your key. Restart the kernel and run from the top.

Reference: UN Comtrade's [official example notebook](https://github.com/uncomtrade/comtradeapicall/blob/main/tests/example%20calling%20functions%20-%20notebook.ipynb) demonstrates its `comtradeapicall` helper package. This project uses `requests` directly to follow the course approach; that helper package is not required.

The local `.env` format is:

```dotenv
UN_COMTRADE_API_KEY=
```

Fill in your primary or secondary Comtrade key locally. Do not use both. The notebook loads this file, or prompts securely when no key is available. Existing environment values take precedence; restart the kernel after changing the file.

Never commit or include `.env` in the submission ZIP. Git ignore rules do not protect a file that was already tracked.

Run the notebook from the top and save the outputs. A complete pilot should contain two records with matching query dimensions and a consistent HS edition. HTTP 401/403 requires checking account/key access; HTTP 429 indicates a rate limit. Do not publish credentials or request headers when asking for help.

After validation, a saving cell creates a dated JSON snapshot under `data/`. It contains raw records, query settings, and download timestamps, but no headers or credentials. Existing snapshots are not overwritten. Saving local JSON does not replace loading and transforming data inside MongoDB.

### Option B: reproduce from supplied data

No Comtrade registration or key is needed for snapshot mode:

1. Obtain the snapshot supplied with the project and keep it in `data/`.
2. Change `DATA_SOURCE` to `"snapshot"` and set `SNAPSHOT_PATH` to that exact dated filename.
3. Restart the kernel and run the notebook from the top.

The notebook skips key loading and API requests, reads the raw records and extraction log, checks the query scope, and applies the same pilot validation. It does not overwrite the supplied snapshot or silently fall back to the API. Future MongoDB steps will still need an appropriate database connection.

The first snapshot does not exist until the API extraction, validation, and saving cells have run. The path printed by the saving cell is the one to use for reproduction. JSON is suitable here because the raw API data uses JSON types; future MongoDB BSON exports may need Extended JSON to preserve dates and ObjectIds.

### Submission and data distribution

The assignment text supplied for this project requires the PDF, notebook, and images in a ZIP; it does not explicitly forbid dataset files or confirm whether additional files are accepted. Check the course's upload guide or ask the professor before assuming that a snapshot is accepted as an extra file.

UN Comtrade has specific [use and re-dissemination terms](https://uncomtrade.org/docs/re-dissemination-of-data/), not an unrestricted open-data license. Check the applicable exceptions and attribution requirements for the intended distribution. A small assessment snapshot and a public raw-data repository are different uses. Do not publish a larger raw dataset without checking the terms.

If accepted, include the selected dated snapshot, `.env.example`, `requirements.txt`, and these instructions alongside the required files. Exclude `.env` and unused snapshots. Preserve the exact snapshot used for the submitted results and identify its filename in the notebook.

## Check validation without API access

```bash
python -m unittest -v test_pilot_validation
```

These tests use synthetic records and do not read `.env`, make API requests, or write to MongoDB.

## Render the PDF

Install Quarto and a working TeX distribution separately if not already available. Quarto's standard PDF workflow can use TinyTeX.

After running and saving the notebook:

```bash
quarto render "UN_commodity.ipynb" --to pdf --no-execute
```

Rendering uses saved outputs and does not execute the API cells. Regenerate the PDF after notebook changes; an existing PDF may be an older preview. Inspect the final tables, images, code layout, and page breaks before submission.