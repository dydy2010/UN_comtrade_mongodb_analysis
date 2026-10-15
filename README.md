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

Create a local `.env` file beside the notebook:

```dotenv
UN_COMTRADE_API_KEY=
```

Fill in your primary or secondary Comtrade key locally. Do not use both. The notebook loads this file, or prompts securely when no key is available. Existing environment values take precedence; restart the kernel after changing the file.

Never commit or include `.env` in the submission ZIP. Git ignore rules do not protect a file that was already tracked.

Run the notebook from the top and save the outputs. A complete pilot should contain two records with matching query dimensions and a consistent HS edition. HTTP 401/403 requires checking account/key access; HTTP 429 indicates a rate limit. Do not publish credentials or request headers when asking for help.

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