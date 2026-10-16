# Project guidance

- Scope: Swiss annual merchandise exports to World, 2022–2025, HS two-digit chapters 01–97 excluding reserved 77. Comtrade reporter code is 757. Require returned edition H6; expect 96 chapters per year and 384 unique year/chapter records. Exclude partial-year 2026 data from annual comparisons.
- Extraction, validation, and snapshot saving are implemented. MongoDB loading and analysis are not yet implemented; do not present planned findings as results.
- Run validation tests from this folder with `python -m unittest -v test_pilot_validation test_india_exploration`. Tests use synthetic data and do not need credentials.
- Install notebook dependencies using this folder's `requirements.txt`, not a parent workspace manifest.
- Render saved notebook outputs with `quarto render "UN_commodity.ipynb" --to pdf --no-execute`. Quarto and TeX are separate system dependencies.
- Do not read, print, commit, or package `.env` secrets. Do not execute authenticated API cells unless requested.
- The notebook supports explicit `api` and `snapshot` modes. Keep snapshot mode network-free and credential-free. Dated raw snapshots belong in Git-ignored `data/`; do not publish them without checking source redistribution terms.
- `India_HS2710_exploration.ipynb` is an unrelated side project for India 2025 HS 2710 imports. Its public-data query requires no key, exports Excel and a chart, and must not modify Swiss settings. Keep World totals separate from partner rows.
- Course guidance and the broader plan live in the parent workspace when available: `../txt course content/Copy_paste_course_content.md` and `../PROJECT_PLAN.md`. Follow the notebook's course references; explain extensions beyond course examples.
