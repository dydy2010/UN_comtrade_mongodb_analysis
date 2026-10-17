# MongoDB Personal Project Plan

## Project goal

Build a reproducible Jupyter Notebook that downloads public data, loads it into MongoDB, cleans and models it, and answers interesting questions using aggregation pipelines.

**Approach:** follow the course template and use beginner-friendly Python and PyMongo syntax.

**Nested sets:** optional. Decide after inspecting the dataset, rather than choosing data only to force this technique.

### Course reference rule for future AI assistance

**If unsure about course-related syntax, coding style, data modeling, operators, or an implementation approach—or if earlier conversation context has been forgotten—read the relevant sections of [Copy_paste_course_content.md](txt%20course%20content/Copy_paste_course_content.md) before proposing or writing code. Do not rely on memory or assume what the course teaches.**

- The notes are stored at `txt course content/Copy_paste_course_content.md`, relative to this project folder.
- Start with the relevant topic heading, such as Data Model Design, Updating Collection Structure, Query an Array of Embedded Documents, Query for Null or Missing Fields, or Tree Structures.
- Prefer the course’s simple Python/PyMongo patterns and explain both the syntax and the logic at a beginner level.
- Distinguish mongosh examples from Python examples; translate syntax correctly rather than copying JavaScript directly into the notebook.
- The course is the starting point, not a restriction. Use additional techniques when they genuinely help, and explain why they are needed and how they relate to the course.
- If an example is outdated, ambiguous, or incorrect, verify the behavior in current official documentation and explain the correction instead of reproducing it blindly.
- Follow the assignment requirements when choosing scope; a topic appearing in the notes does not make it compulsory.

### Requirements versus learning goals

- **Assignment requirements:** scripted download of public, free, non-Kaggle data; loading into a MongoDB cluster from a notebook; modeling and cleaning in MongoDB; in-depth aggregation analysis; at least two images including a class diagram; PDF and notebook submission with images in a ZIP.
- **Assignment recommendation:** try to use at least four aggregation pipelines, counting transformation, cleaning, and analysis pipelines.
- **Suggested scope, not extra requirements:** 3–4 research questions and around 5–6 meaningful pipelines overall. Adjust these to the dataset; depth and clear reasoning matter more than an arbitrary count.
- **Optional extensions:** multiple sources, nested sets, geospatial analysis, text search, and a small indexing demonstration. Do not attempt all of them.

The three grading areas have equal weight: PDF quality, transformation complexity, and aggregation sophistication. The project contributes 80% of the NoSQL Lab grade.

---

## 1. Find and evaluate a dataset

- [ ] Shortlist 2–3 topics you find interesting.
- [x] Find public, free data available through an API or downloadable file URL.
- [x] Exclude Kaggle datasets.
- [x] Test a small download before committing to a source.
- [x] Inspect a few records and the API documentation.

### Dataset selection checklist

| Check | What to look for |
|---|---|
| Access | Free access, manageable authentication and rate limits |
| Format | Prefer JSON with nested objects or arrays |
| Questions | Enough information for 3–4 meaningful research questions |
| Cleaning | Opportunities to handle missing values, types, duplicates, or inconsistent structures |
| Scope | A manageable download, not necessarily a large dataset |
| Documentation | Understandable fields, units, dates, and identifiers |
| Hierarchy | A genuine category tree, if considering nested sets |

- [x] Review the source’s usage terms and attribution requirements. Recheck the applicable terms before sharing raw data.
- [x] Define a bounded scope, such as a date range, region, or category, and explain its limitations.

**Output (completed):** UN Comtrade was selected for its documented public API and useful annual trade measures. The V1 bounded scope is Switzerland’s 2022–2025 exports to World by HS chapter, with four research questions recorded in the notebook.

**V2 scope (planned):** keep V1 as the baseline and create `mongodb_commodity/UN_Comtrade_V2.ipynb` for Switzerland’s annual exports in 2023–2025. V2 will first download partner totals. After an authorized MongoDB stage, rank the top 30 partner areas by cumulative all-commodity value and then the top 50 HS4 headings by value within those partners using aggregation pipelines. Keep other reporting dimensions at their aggregate selections.

---

## 2. Create the notebook structure

Follow the assignment template:

```text
1 Summary
2 Requirements & Configuration
3 ELT Process
  3.1 Database Setup
  3.2 Extract
  3.3 Load
  3.4 Transform and Data Quality
  3.5 Data Structure
4 Data Analysis
  4.1 Research Question 1
  4.2 Research Question 2
  4.3 Research Question 3
5 Conclusions
6 Learnings
7 Sources
```

- [ ] Use numbered headings and a table of contents.
- [ ] Write brief Markdown explanations before code.
- [ ] Test PDF export early with a table and an image.

**Practical focus:** clear communication matters as much as technical work; PDF quality contributes one-third of the project grade.

The template’s older Jupyter extensions and PDF instructions may not match the installed environment. Test an available export method before relying on them.

### PDF workflow: try Quarto first

The assignment lists Quarto as an option, not a requirement. Our intended workflow is:

**Write and execute the Jupyter Notebook → Save its outputs → Render with Quarto → Inspect the PDF**

- [ ] Check whether Quarto is installed; it is a separate publishing tool, not included automatically with Jupyter.
- [ ] For Quarto’s standard LaTeX PDF workflow, check for a working TeX installation. If needed, install TinyTeX using `quarto install tinytex`.
- [ ] Test a small notebook containing a heading, paragraph, aggregation result table, and local image before formatting the whole project.
- [ ] Configure the title, author, table of contents, and section numbering in notebook-level Quarto metadata.
- [ ] Use either automatic section numbering or manually numbered headings, not both.
- [ ] Keep selected aggregation code visible where it helps demonstrate the work; hide setup or verbose download code from the PDF if useful. Keep all required code in the submitted notebook.
- [ ] Check table rendering early. Wide Pandas tables may need fewer columns, shorter labels, or an export-friendly representation.

Once the real notebook filename is chosen, the basic render command is:

```bash
quarto render "project.ipynb" --to pdf
```

`project.ipynb` is a placeholder. Quarto normally uses saved outputs when rendering an existing notebook unless execution is requested. Restart the kernel, run all cells successfully, and save before the final render; avoid accidentally rerunning API downloads or database writes through an additional execution step.

**Fallback:** if Quarto setup or layout becomes a distraction, use the assignment’s browser **Print → Save as PDF** approach. Check print preview, headings, tables, images, and unnecessary interface elements. Quarto does not replace the requirement to submit the notebook and images.

---

## 3. Configure Python and MongoDB

- [ ] Use the course’s main packages: `requests`, `pymongo`, and `pandas`.
- [ ] Add other packages only when needed.
- [ ] Keep API settings and database names together.
- [ ] Read database credentials from outside the notebook, such as an environment variable.
- [ ] Connect to the MongoDB cluster and verify the connection with `client.admin.command("ping")`; creating `MongoClient` alone does not prove the connection works.
- [ ] Select clearly named project collections.
- [ ] Record the Python, package, and MongoDB server versions used.

### Suggested collection structure

```text
project_database
├── records_raw
├── records_clean
└── categories       ← only if the data needs a separate hierarchy
```

**Course concepts:** `MongoClient`, databases, collections, documents, and `_id`.

Raw and clean collections are a recommended design, not a requirement. Select the database and collections before loading, but make the main modeling decisions after inspecting sample data. Selecting `client[DB_NAME]` alone does not create a stored database.

**Output:** a working connection without credentials appearing in code, outputs, or the submitted ZIP.

---

## 4. Extract data using notebook code

**Request → Check response → Parse JSON → Inspect → Repeat if needed**

- [ ] Start with one small request.
- [ ] Check HTTP errors before parsing the response.
- [ ] Use a request timeout.
- [ ] Handle pagination if the API returns batches.
- [ ] Respect rate limits.
- [ ] Verify the returned JSON structure and stop with a clear error if an expected field is absent.
- [ ] Record the source, download date, filters, and coverage.
- [ ] Inspect nested objects, arrays, and source identifiers.

**Coding style:** simple loops, clear variable names, and small functions where they avoid repetition.

**Output:** downloaded records with a documented scope.

---

## 5. Load the raw data into MongoDB

- [ ] Preserve the source fields with minimal changes.
- [ ] Load records using `insert_many()` or another suitable PyMongo operation.
- [ ] Check the inserted document count.
- [ ] Inspect sample documents using `find_one()` and limited `find()` queries.
- [ ] Decide how repeated notebook runs will avoid accidental duplicate imports.

Choose a simple rerun policy before the first full import. For example, use stable source IDs with `replace_one(..., upsert=True)` so the same records are replaced rather than appended. This is a modest extension of the course’s replacement/upsert examples. It does not remove records that disappear from the source, so distinguish an incremental refresh from a complete snapshot.

For larger downloads, load manageable batches. Avoid putting the entire dataset into one document; MongoDB has a 16 MiB document-size limit.

**Course concepts:**

- Python dictionaries → BSON documents.
- Python lists → arrays.
- Unique `_id` values.
- `insert_many()`, `find_one()`, `find()`, and `count_documents()`.

**Important:** do not use `drop()` or `delete_many({})` as an unexplained automatic reset.

**Output:** a raw collection that can be checked against the downloaded data.

---

## 6. Profile and clean the data inside MongoDB

### Inspect first

- [ ] Count missing and explicit null values.
- [ ] Check field types.
- [ ] Identify duplicate source IDs or other meaningful duplicate keys.
- [ ] Check invalid values, dates, and numeric ranges.
- [ ] Inspect inconsistent category names or units.
- [ ] Define what makes two records duplicates and which record should be retained, such as the most recently updated one.

### Transform only where justified

| Task | Techniques |
|---|---|
| Inspect missing fields | `$exists`, `$type`, queries using `None` |
| Convert numeric strings | `$toInt` for whole numbers; `$convert` to an appropriate numeric type when needed |
| Convert date strings | `$dateFromString`, with explicit format/timezone handling where needed |
| Add derived fields | `$addFields` / `$set` |
| Remove unnecessary fields | `$unset` or `$project` |
| Detect duplicates | `$group`, `$sum`, `$match` |
| Save transformed documents | `$out` to a dedicated clean collection |
| Update stored fields | `update_many()` with an update pipeline |

`$convert` would be a small extension beyond the examples if needed. Explain any additional syntax.

Do not convert decimal measurements or prices to integers: this can discard fractional values. Use a suitable type, such as double for measurements or Decimal128 where exact decimal arithmetic is needed. If conversions fail, count and inspect the failures rather than silently treating them as valid values.

**Before cleaning → Explain the rule → Transform → Check again**

Do not invent errors just to demonstrate cleaning. A check that finds no problems is still useful evidence.

Remember that a query such as `{"field": None}` matches both explicit null values and missing fields. Use `$type` or `$exists` to distinguish them.

Keep the raw collection unchanged. `$out` replaces the destination collection if it already exists, so use a dedicated clean collection deliberately.

Ordinary `aggregate()` results do not modify stored documents. Persist transformations with `$out` or an appropriate update operation. Use separate stages when a calculated field depends on another field created earlier in the pipeline.

The course duplicate-removal example collects IDs using `$addToSet`, whose order is unspecified. Do not assume its first ID is the earliest or best record; use an explicit retention rule.

**Output:** clean data and a compact before/after quality summary.

---

## 7. Explain and visualize the data model

- [ ] State what one document represents.
- [ ] Explain important fields and BSON types.
- [ ] Explain which information is embedded and which is referenced.
- [ ] Use dot notation to inspect nested fields and `$elemMatch` where conditions must match the same array element.
- [ ] Include a **class diagram** matching the actual cleaned structure.
- [ ] Include a second image, such as an ELT dataflow diagram.

### Optional: nested sets

Only include them if the dataset has a genuine, suitable hierarchy.

**Identify hierarchy → Assign `left`/`right` boundaries → Validate → Query branches → Analyse related records**

Explain both the advantage for branch reads and the difficulty of updates. Distinguish immediate children from all descendants.

Prefer a relatively stable tree. Check `left < right`, unique traversal boundaries, and valid parent/descendant containment. If Python is used to assign traversal numbers, explain that step and still demonstrate meaningful MongoDB transformations and branch analysis. Do not assume nested sets are always faster than other tree models.

Nested sets represent a tree through number ranges. They are different from embedded JSON documents, although both can be used in the same project.

**Output:** two readable images and a justified model—not complexity for its own sake.

---

## 8. Answer research questions with aggregation pipelines

Aim for **around 5–6 meaningful pipelines overall**, including transformation and quality-check pipelines. The assignment suggests at least four.

Choose analyses based on the available fields:

The core group/array techniques and conditional reshaping follow the supplied notes. `$lookup` and some date expressions may require additional documentation; they are optional extensions rather than prerequisites.

| Analysis idea | Possible operators or features |
|---|---|
| Compare groups | `$match`, `$group`, `$sum`, `$avg`, `$sort` |
| Analyse array contents | `$unwind`, `$group`, `$sort`, `$limit` |
| Compare time periods | Date expressions, `$group`, `$sort` |
| Create meaningful classifications | `$cond`, `$addFields`, `$group` |
| Reshape dynamic object keys | `$objectToArray`, `$arrayToObject` |
| Combine related collections | `$lookup`, only if justified |
| Analyse locations | GeoJSON, `2dsphere`, `$geoNear`, only if useful |

### Structure each analysis

**Question → Why it matters → Pipeline → Result → Interpretation → Limitation**

- [ ] Explain what each stage does.
- [ ] Use Pandas mainly to display aggregation results and support charts.
- [ ] Check sample sizes alongside averages.
- [ ] Check whether `$unwind` changes what is being counted.
- [ ] Decide how missing or empty arrays should be handled; `$unwind` excludes them by default.
- [ ] Do not turn missing values into zero without justification.
- [ ] Avoid claims unsupported by the data, such as inferring sales from catalogue listings.
- [ ] Use a suitable denominator when reporting percentages and a stable secondary sort for tied rankings.

Use varied stages and operators where they serve the analysis, rather than splitting one operation into several pipelines to increase the count. A `find()` query is useful inspection, but is not an aggregation pipeline.

### Coding style

- Use readable Python dictionaries and lists for queries and pipelines.
- Use small, sequential notebook cells.
- Prefer clear variable names and simple loops.
- Use small functions when they avoid repetition.
- Avoid unnecessary classes or complicated abstractions.
- Explain operators that go beyond the supplied course examples.

Use Python/PyMongo syntax in notebook code: quoted operator keys, `True`/`False`/`None`, and methods such as `find_one()`, `update_many()`, and `create_index()`. The notes also contain mongosh examples, which use different syntax. Use `count_documents()` rather than older `.count()` examples.

**Output:** evidence-based answers, not just tables of query results.

---

## 9. Add practical reliability checks

- [ ] Optionally choose an index that supports a real query.
- [ ] If adding an index, explain why it is relevant.
- [ ] Optionally inspect a query with `explain()`.
- [ ] Validate that rerunning on the same source snapshot does not accumulate duplicates; explain count changes when live source data changes.
- [ ] Keep raw data separate from cleaned results.
- [ ] Avoid printing credentials or sensitive configuration.

**Course connection:** indexes, query performance, CRUD operations, and data integrity.

A small dataset does not need a large indexing exercise. One justified example is enough for learning. Do not claim a performance improvement without evidence. Geospatial queries have specific index and stage requirements that should be checked if selected.

---

## 10. Write, verify, and submit

### Writing

- [ ] Complete the summary after obtaining results.
- [ ] Answer each research question in the conclusion.
- [ ] Discuss coverage, missing data, and other limitations.
- [ ] Describe your actual learnings.
- [ ] Cite data sources and documentation.

### Verification and packaging

- [ ] Restart the kernel and run the notebook from top to bottom.
- [ ] Confirm there are no errors or hidden dependencies.
- [ ] Include setup instructions for required environment variable names and packages, but never their secret values.
- [ ] Check that diagrams match the implemented model.
- [ ] Use portable relative image paths and check that images also appear in the exported PDF.
- [ ] Export the PDF and inspect every page.
- [ ] Check tables, image readability, headings, and page breaks.
- [ ] Include the PDF, executed notebook, and required images in the ZIP.
- [ ] Exclude credentials and unrelated files.
- [ ] Follow the ILIAS upload instructions.

**Deadline stated in the assignment: 30 October 2026, 23:55 CET.**

## Reference material

- Assignment and project template supplied in the conversation.
- [Local course notes](txt%20course%20content/Copy_paste_course_content.md): Data Model Design; Updating Collection Structure; querying nested documents and arrays; null/missing fields; duplicate detection; type conversion; Group and Push; and Tree Structures.
- Consult current official MongoDB, PyMongo, and API documentation for optional extensions or behavior that depends on software versions.

---

## Immediate next step

**[x] Find and test a few datasets.** Completed: UN Comtrade was selected and a Swiss pilot extraction was tested.

**V2 progress:** the notebook has a validated 673-record partner snapshot for 2023-2025. The user has connected to Atlas, loaded the partner totals, and ranked the top 30 partner areas in MongoDB. Code `490`, `Other Asia, nes`, remains eligible with its official label. V1 remains unchanged.

**Next step: execute and inspect the new HS4 stage.** The code now supports 90 partner-year requests, validation, a separate dated snapshot, stable-ID loading into `hs4_raw`, and MongoDB top-50 ranking. These new cells remain unexecuted and require explicit opt-in. Offline tests use synthetic records and mocked services; they do not prove live API success.

**After verifying HS4 results:** implement the nested clean collection, complete research-question pipelines, and write conclusions. Keep source collections unchanged. Do not infer zero trade from absent observations or use the discovery snapshot's placeholder weights for volume analysis.
