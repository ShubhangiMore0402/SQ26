# QDArchive

## Part 1: Data Acquisition

This repository contains the data acquisition pipeline for **Part 1** of the QDArchive seeding project. The goal is to find, download, and catalogue publicly available qualitative research projects with a focus on QDA (Qualitative Data Analysis) files from Zenodo and Harvard Dataverse.

All metadata is stored in a local SQLite database. Downloaded files follow a standardised folder structure. No data is transformed at download time — raw values are preserved and quality issues are flagged separately.

---

## Repository structure

```
.
├── qdarchive_part1.ipynb       # Main notebook: queries, schema, pipeline
├── qdarchive_metadata.db       # SQLite database (generated and included)
├── downloads/                  # Sample downloaded datasets (small subset for GitHub)
│   ├── zenodo/
│   │   └── <record_id>/
│   │       └── small files (e.g. .txt, .csv, .json)
│   └── harvard-dataverse/
│       └── <DVN_ID>/
│           └── small files (fallback download if no small files available)
├── export/                     # CSV exports of all DB tables
│   ├── projects.csv
│   ├── files.csv
│   ├── keywords.csv
│   ├── person_role.csv
│   └── licenses.csv
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Setup

**Requirements:** Python 3.10+

```bash
# 1. Clone the repo
git clone <your-repo-url>
cd <repo-folder>

# 2. Create and activate a virtual environment (recommended)
python -m venv .venv
source .venv/bin/activate        # macOS / Linux
.venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt
```

---

## Running the notebook

Open `qdarchive_part1.ipynb` in Jupyter or VS Code and run cells top to bottom.

```bash
jupyter notebook qdarchive_part1.ipynb
# or
jupyter lab
```

---

## Execution modes

### 1. Dry-run mode (default, safe)

```python
DRY_RUN = True
```

* No files are downloaded
* Metadata is inserted into SQLite
* File downloads are simulated

---

### 2. Full pipeline mode

```python
DRY_RUN = False
```

* Downloads all available files (may be large and slow)
* Not recommended for GitHub submission due to file size limits

---

### 3. Small sample download mode (used for submission ✅)

A dedicated helper function is provided:

```python
run_small_download_sample()
```

This mode:

* Downloads a **very small subset of files**
* Prioritises small formats (`.txt`, `.csv`, `.json`, etc.)
* Falls back to any available file if small ones are not present
* Enforces a size cap (~10 MB)
* Guarantees **at least one file is downloaded per repository (if available)**

👉 This ensures the `downloads/` folder is:

* valid
* non-empty
* GitHub-safe

---

## SQLite schema

| Table          | Purpose                                              |
| -------------- | ---------------------------------------------------- |
| `repositories` | Master list of repositories                          |
| `projects`     | One row per discovered research project              |
| `files`        | One row per file; tracks download status             |
| `keywords`     | Raw keyword strings (no cleaning at ingestion)       |
| `person_role`  | Contributors with roles (`AUTHOR`, `UPLOADER`, etc.) |
| `licenses`     | Raw license strings                                  |

Download status values:
`SUCCEEDED` · `FAILED_SERVER_UNRESPONSIVE` · `FAILED_LOGIN_REQUIRED` · `FAILED_TOO_LARGE`

---

## Search queries

### Zenodo

| Query                                                                             | Rationale                        |
| --------------------------------------------------------------------------------- | -------------------------------- |
| `qdpx`                                                                            | Direct hit on REFI-QDA extension |
| `mx24 OR mqda OR mx22`                                                            | MaxQDA project formats           |
| `nvp OR nvpx OR atlasproj OR hpr7`                                                | NVivo + ATLAS.ti                 |
| `"qualitative research data" AND (interview OR transcript)`                       | Explicit qualitative datasets    |
| `metadata.title:(qualitative) AND metadata.title:(MAXQDA OR NVivo OR "ATLAS.ti")` | Title-field targeting            |
| `"interview study" AND (transcript OR coding OR thematic)`                        | Methodology-based search         |

### Harvard Dataverse

| Query                                                            | Rationale                |
| ---------------------------------------------------------------- | ------------------------ |
| `qdpx`                                                           | REFI-QDA extension       |
| `MAXQDA OR NVivo OR "ATLAS.ti"`                                  | Tool-based search        |
| `"qualitative" AND "interview" AND "transcript"`                 | Interview-based datasets |
| `"thematic analysis" OR "grounded theory" OR "content analysis"` | Methodology keywords     |

---

## Open licenses accepted

Only open-license datasets are downloaded (Zenodo pipeline).

Examples:

`CC BY 4.0` · `CC BY-SA 4.0` · `CC BY-NC 4.0` · `CC BY-ND 4.0` · `CC0 1.0` · `ODbL 1.0` · `ODC-By` · `PDDL`

Datasets without a license are skipped.

---

## Known data quality issues

1. **Keyword format inconsistency** — some keywords appear as comma-separated strings.
2. **Missing license** — such records are skipped.
3. **Zenodo file list incomplete** — some records require a secondary API call.
4. **Language missing (Dataverse)** — not available at search level.
5. **Cross-repository duplicates** — resolved via DOI in later stages.

---

## Notes on downloaded datasets

* Only a **small representative subset** of files is included in `downloads/`
* This is intentional due to:

  * GitHub file size limits
  * large dataset sizes in repositories
* The pipeline supports full-scale downloads when `DRY_RUN = False`

---
## License

Code in this repository: MIT
Downloaded research data: governed by individual dataset licenses stored in the database.
