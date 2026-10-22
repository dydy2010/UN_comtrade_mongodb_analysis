# UN Comtrade MongoDB Analysis

This course project studies Switzerland's goods exports to 30 selected partner areas during 2023-2025.
V3 is the final version; all setup, reproduction, and rendering instructions below apply to V3.
The final report uses MongoDB aggregations to answer three questions:

1. Which partner areas lead in 2025, and how did their shares change since 2023?
2. Which ten HS4 headings have the highest 2025 export value across the selected areas?
3. How do the five largest 2025 partner areas differ in their HS2 commodity structure?

## Start here

1. Read [the V3 PDF report](UN_Comtrade_V3.pdf) for the results, methods, and model diagrams.
2. Open [the V3 notebook](UN_Comtrade_V3.ipynb) for saved outputs and detailed reproduction instructions.
3. Follow the steps below only if you want to validate inputs or recalculate results.

Viewing saved outputs requires neither MongoDB nor a UN Comtrade API key.
The PDF includes the schema comparison and the final UML diagram.
The `screenshots/` folder contains the diagram files that the notebook uses.

## Data and analytical scope

| Item | Scope |
|---|---|
| Source | UN Comtrade |
| Reporter | Switzerland, Comtrade code `757` |
| Flow and frequency | Annual goods exports, `X`, 2023-2025 |
| Classification | Original H6, the HS 2022 edition |
| Partner selection | Top 30 non-World areas by cumulative all-commodity value across 2023-2025 |
| Product detail | Standard four-digit HS headings, requested with `cmdCode="AG4"` |
| Measure | `primaryValue`, in current US dollars |
| Other query dimensions | `partner2Code=0`, `customsCode="C00"`, `motCode=0` |

The same 30 partner areas remain in scope for all three years.
The selection includes code `490`, `Other Asia, nes`, under its official partner-area label.
This retrospective selection uses all three years, not a separate top-30 ranking for each year.

The discovery dataset contains all-commodity `TOTAL` rows, not HS2 product detail.
The HS4 extraction contains 90 partner-year batches and 72,459 raw observations.
The H6 reference check excludes 90 nonstandard `9999` observations from the clean model.
The verified `trade_clean` collection contains 72,369 standard HS4 observations.

HS2 summaries group the full standard HS4 detail by its first two code digits.
The earlier top-50 heading drill-down contains 4,235 observations; it does not define the main analytical dataset.

Shares describe standard-heading value within the selected 30 areas, not all Swiss exports.
Missing combinations remain absent; the analysis does not fill them with zero.
Current-dollar changes do not establish physical-volume growth, profitability, or causation.
Coverage checks compare HS4 sums with frozen all-commodity controls without rescaling differences.

## Frozen snapshots

V3 uses these exact files:

| File under `data/` | Observation count | Purpose |
|---|---:|---|
| `v2_hs4_snapshot_20261017T094752738553Z.json` | 72,459 | HS4 observations, H6 reference, partner ranking, control totals, and extraction logs |
| `v2_partner_totals_20261016T143600945247Z.json` | 673 | Historical all-commodity discovery observations and extraction logs |

Together, the two observation arrays contain 73,132 records.
The HS4 snapshot also preserves 90 partner-year control totals from discovery.
The `v2_` filenames identify the original extraction, not the final report version.
V3 reuses these frozen inputs without a new API request or reference download.
Later UN Comtrade downloads may contain revisions.

## Download the data

The approximately 106 MB HS4 JSON uses Git LFS.
The smaller discovery JSON uses ordinary Git storage.
Install Git and [Git LFS](https://git-lfs.com/) before cloning.

```bash
git lfs install
git clone https://github.com/dydy2010/UN_comtrade_mongodb_analysis.git
cd UN_comtrade_mongodb_analysis
git lfs pull
```

For an existing clone, run `git lfs pull` from its repository folder.
Without LFS, the HS4 path may contain a short pointer instead of the actual JSON.
Do not use that pointer as notebook input.

As an alternative, download the actual snapshot from its GitHub file page.
Place it under `data/` with the exact filename above.
Check that the HS4 file contains JSON rather than a line starting with `version https://git-lfs.github.com/spec/v1`.
Do not assume a GitHub source ZIP includes the full LFS object.
GitHub's LFS storage and download limits can affect access.

## Set up Python

Run these commands from the repository folder on macOS or Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m notebook
```

On Windows, activate the environment with `.venv\Scripts\activate` in Command Prompt.
Open `UN_Comtrade_V3.ipynb` and select this environment as its kernel.
Keep the kernel's working directory at the repository root.
Use this repository's `requirements.txt`, not a parent folder's dependency file.

## Choose a reproduction mode

Work on a notebook copy to preserve the submitted outputs.
Choose `RUN_MODE` in the configuration cell before execution.
Restart the kernel, then run all cells in order.
Do not change modes halfway through a session.

| Mode | Requirements | Behavior |
|---|---|---|
| `offline` | Both actual JSON snapshots | Validate frozen inputs and coverage; skip MongoDB and its analyses |
| `verify` | Snapshots, own MongoDB URI, matching existing collections | Recalculate and verify without database writes |
| `rebuild` | Snapshots, own MongoDB URI, explicit write approval, isolated database | Create missing collections, verify stored data, and recalculate results |

All modes keep `DATA_SOURCE = "snapshot"` and `HS4_DATA_SOURCE = "snapshot"`.
V3 rejects live API settings.
No V3 reproduction mode needs a UN Comtrade API key.
The notebook derives later operation flags from the selected mode.
Do not enable those flags individually.

### 1. Validate inputs offline

Keep the supplied defaults:

```python
RUN_MODE = "offline"
DB_NAME = "swiss_trade_v2"
ALLOW_DATABASE_WRITES = False
```

This mode needs no credentials, network access, or MongoDB installation.
It validates source data but does not reproduce MongoDB analysis results in Python.
Execution replaces cached outputs in your copy, including cells that skip analysis.
Use the original saved notebook or PDF to read the completed results.

### 2. Verify an existing database

Provide your own `MONGODB_URI` through your environment or a private `.env` file.
The tracked `.env.example` contains empty placeholders, not credentials.
Use a read-only MongoDB account when possible.

```python
RUN_MODE = "verify"
DB_NAME = "your_existing_database"
ALLOW_DATABASE_WRITES = False
```

The database must contain matching `partner_totals_raw`, `hs4_raw`, and `trade_clean` collections.
The notebook stops on missing or different data instead of repairing collections.
The original saved run used `swiss_trade_v2`; your database name may differ.

### 3. Rebuild in a separate database

Use your own MongoDB instance and an account that can create collections and run `$out`.
Choose an isolated database whose name starts with `swiss_trade_v3_reproduction`.

```python
RUN_MODE = "rebuild"
DB_NAME = "swiss_trade_v3_reproduction_professor"
ALLOW_DATABASE_WRITES = True
```

This mode loads missing raw collections and saves a missing clean collection after preview checks pass.
It verifies existing collections without replacing them.
Mismatches stop execution.
Do not run another process against the same reproduction database during a rebuild.
After a successful rebuild, use `verify` mode with write approval disabled for later runs.

Never commit `.env`, API keys, MongoDB URIs, or request headers.
The ignore rules exclude `.env` and allow only the two named snapshots under `data/`.

## Render the report

Install [Quarto](https://quarto.org/) and a working TeX distribution separately.
The notebook's PDF configuration uses LuaLaTeX and disables automatic TeX package installation.
Run:

```bash
quarto render "UN_Comtrade_V3.ipynb" --to pdf --no-execute
```

This command renders saved outputs without executing notebook code.
It makes no API request or MongoDB connection.
After notebook edits, regenerate the PDF and inspect tables, code, images, and page breaks.

## Data attribution and redistribution

Data source: **UN Comtrade**.
We share the two frozen snapshots free of charge for course reproduction.
The [UN Comtrade redistribution policy](https://uncomtrade.org/docs/re-dissemination-of-data/) exempts free extraction applications offering up to 100,000 records.
The two snapshots contain 73,132 trade observations, below that threshold.
We rely on this small-data exception, not a claim that JSON conversion transforms the source data.

We checked the policy on 22 October 2026.
Its transformation guidance excludes format conversion, filtering, structural changes, and variable renaming as substantial transformations.
We therefore do not describe the raw snapshots or the embedded clean model as automatically unrestricted transformed data.
Calculated shares, growth measures, and aggregations are examples of analytical transformations in the policy.

The United Nations retains copyright in the source data.
See the [data-use policy](https://uncomtrade.org/docs/policy-on-comtrade-data-use/) for citation requirements and the applicable licence agreement.
The [UN Comtrade disclaimer](https://uncomtrade.org/docs/read-me-first-disclaimer/) explains source-data limitations, including confidentiality and differences between detailed sums and totals.
This repository does not grant a separate unrestricted licence over UN Comtrade source observations.
Recheck the policy before adding datasets, expanding redistribution, or charging for access.
Contact `comtrade@un.org` or `subscriptions@un.org` if your intended use remains unclear.

## Earlier versions

Read [V1](UN_commodity.ipynb) for the World-by-HS2 baseline and [V2](UN_Comtrade_V2.ipynb) for original partner discovery and HS4 extraction.
Treat both as historical notebooks; check their API, connection, and write settings before execution.

## Submission

Package the V3 PDF, saved V3 notebook, and required diagram images according to the course instructions.
Keep notebook-relative image paths intact.
Include the actual snapshots only if the course accepts dataset files and its upload limits permit them.
Include `README.md`, `requirements.txt`, and `.env.example` when supplying a reproduction package.
Exclude `.env`, unrelated notebooks, unused snapshots, and LFS pointer files.