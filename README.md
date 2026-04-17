# QDArchive Part 1: Data Acquisition

**Student:** Shubhangi More · ID: 23137504  
**Repositories:** Zenodo (Repo #1) · Harvard Dataverse (Repo #10)  
**Course:** Seeding QDArchive · FAU Erlangen · Winter 2025/26 + Summer 2026

---

The goal of this part is to find, download, and catalogue publicly available qualitative research projects — with a focus on QDA (Qualitative Data Analysis) files from two assigned repositories. Metadata is stored in a SQLite database. Files are saved as-is with no transformations at download time; data quality issues are flagged separately.

---

## Repository structure

```
.
├── 23137504-seeding.ipynb      # Main notebook: queries, schema, pipeline
├── 23137504-seeding.db         # SQLite metadata database (committed to repo)
├── downloads/                  # Downloaded datasets (not committed share via link)
│   ├── zenodo/
│   │   └── <record_id>/
│   │       └── *.qdpx, *.docx, ...
│   └── harvard-dataverse/
│       └── <dataset_id>/
│           └── *.zip or individual files
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Setup

Python 3.10+ required.

```bash
git clone <yhttps://github.com/ShubhangiMore0402/SQ26.git>

python -m venv .venv
source .venv/bin/activate      # macOS / Linux
.venv\Scripts\activate         # Windows

pip install -r requirements.txt
```

---

## Running the notebook

Open `23137504-seeding.ipynb` in Jupyter or VS Code and run all cells from top to bottom.

```bash
jupyter notebook 23137504-seeding.ipynb
```

In **Section 7 (Full Pipeline Run)**, set the mode before running:

```python
DRY_RUN = True   # default- inserts metadata, simulates downloads, no files written
DRY_RUN = False  # actually downloads files to the downloads/ folder
```

Run with `DRY_RUN = True` first to verify queries are returning sensible results, then switch to `False` for the real download.

---

## How the download pipeline works

### Zenodo

Searches the Zenodo REST API (`/api/records`) using a set of queries targeting QDA file extensions and qualitative research keywords. For each result with an open license, all files in the record are downloaded and saved under `downloads/zenodo/<record_id>/`.

### Harvard Dataverse

Searches the Dataverse Search API and then attempts to download files using two strategies in sequence:

1. **Individual file download** : calls `/api/datasets/:persistentId/versions/:latest/files` to get the file list, then downloads each file via `/api/access/datafile/{id}`. This works for datasets hosted directly on Harvard Dataverse.

2. **Dataset zip fallback** : if no individual files are returned (common for datasets cross-listed from ICPSR, DANS, DataONE, etc.), the pipeline falls back to `/api/access/dataset/:persistentId/` which downloads the entire dataset as a single zip file. This is saved as `<dataset_id>.zip` in the project folder.

Both strategies log their outcome per file with a status: `SUCCEEDED`, `FAILED_LOGIN_REQUIRED`, `FAILED_SERVER_UNRESPONSIVE`, or `FAILED_TOO_LARGE`.

---

## SQLite schema

The database file is `23137504-seeding.db` and must be committed to the root of the repository for submission.

| Table | Purpose |
|---|---|
| `repositories` | The two assigned repositories (Zenodo, Harvard Dataverse) |
| `projects` | One row per research project found |
| `files` | One row per file with download status |
| `keywords` | Raw keyword strings, one per row |
| `person_role` | Contributors with role: `AUTHOR`, `UPLOADER`, `OWNER`, `OTHER`, `UNKNOWN` |
| `licenses` | Raw license string as returned by the API |

---

## Search queries

### Zenodo

| Query | Rationale |
|---|---|
| `qdpx` | Direct hit on the REFI-QDA standard extension |
| `mx24 OR mqda OR mx22` | MaxQDA project file formats |
| `nvp OR nvpx OR atlasproj OR hpr7` | NVivo and ATLAS.ti extensions |
| `"qualitative research data" AND (interview OR transcript)` | Projects explicitly describing qualitative data |
| `metadata.title:(qualitative) AND metadata.title:(MAXQDA OR NVivo OR "ATLAS.ti")` | Title-field search using QDA tool names |
| `"interview study" AND (transcript OR coding OR thematic)` | Methodology-based signal words |

### Harvard Dataverse

| Query | Rationale |
|---|---|
| `qdpx` | REFI-QDA standard extension |
| `MAXQDA OR NVivo OR "ATLAS.ti"` | QDA tool names — Dataverse indexes file contents |
| `"qualitative" AND "interview" AND "transcript"` | Interview-based qualitative projects |
| `"thematic analysis" OR "grounded theory" OR "content analysis"` | Qualitative methodology terms |

---

## Known data quality issues

1. **Keyword format inconsistency** — some Zenodo records store multiple keywords as a single comma-separated string rather than separate values. Stored as-is; splitting is deferred to a later step.
2. **Missing license field** — some records have no license. These are recorded with `NULL` in the licenses table; no records are skipped.
3. **Zenodo file list incomplete in search response** — the search endpoint does not always include the `files` array. A secondary call to `/api/records/{id}` may be needed.
4. **Language field absent in Harvard Dataverse** — not available at search-result level; stored as `NULL`.
5. **Cross-repository duplicates** — some datasets appear on both Zenodo and Harvard Dataverse. DOI-based deduplication is handled in the Part 2 merge step.
6. **Harvard Dataverse cross-listed datasets** — many results in the HDV search are datasets hosted on third-party repositories (ICPSR, DANS, DataONE). The individual file list API returns empty for these, which is why the zip fallback strategy exists.

---

## Submission

The database `23137504-seeding.db` is committed to the root of this repository. Downloaded files are shared via a separate link (too large for GitHub).

```bash
git add 23137504-seeding.db
git commit -m "part-1: add seeding database"
git tag part-1-release
git push origin main --tags
```

---

## License

Code: MIT  
Downloaded research data: see individual project licenses stored in the `licenses` table.
