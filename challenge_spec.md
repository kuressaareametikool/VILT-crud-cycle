# 🏆 CLI Challenge: Estonian Business Registry Importer

**Difficulty:** Intermediate–Advanced  
**Estimated Time:** 4–8 hours  
**Language:** Any (Python, Go, Rust, Node.js, etc.)  
**Data Source:** [e-äriregistri avaandmed](https://avaandmed.ariregister.rik.ee/et/avaandmete-allalaadimine)

---

## Background

The Estonian Business Registry (äriregister) publishes open data about all registered companies, non-profits, foundations, and state institutions. The dataset is updated **daily** and is available as a downloadable CSV (zipped). Your task is to build a command-line tool that downloads, parses, and imports this data into a database — as fast, correctly, and robustly as possible.

---

## The Data

**Download URL (CSV):**
```
https://avaandmed.ariregister.rik.ee/sites/default/files/avaandmed/ettevotja_rekvisiidid__lihtandmed.csv.zip
```

The ZIP contains a semicolon-delimited CSV with the following fields:

| Field | Description |
|---|---|
| `nimi` | Company name |
| `ariregistri_kood` | Registry code (unique identifier) |
| `ettevotja_oiguslik_vorm` | Legal form code |
| `ettevotja_oigusliku_vormi_alaliik` | Legal form subtype |
| `kmkr_nr` | VAT number |
| `ettevotja_staatus` | Status code |
| `ettevotja_staatus_tekstina` | Status as text |
| `ettevotja_esmakande_kpv` | Date of first registration |
| `ettevotja_aadress` | Full address |
| `asukoht_ettevotja_aadressis` | Location within address |
| `asukoha_ehak_kood` | EHAK location code |
| `asukoha_ehak_tekstina` | EHAK location as text |
| `indeks_ettevotja_aadressis` | Postal index |
| `ads_adr_id` | ADS address ID |
| `ads_ads_oid` | ADS object ID |
| `ads_normaliseeritud_taisaadress` | ADS normalised full address |
| `teabesysteemi_link` | Link to the e-registry entry |

The dataset contains **~300,000+ records** and is around **60–100 MB** uncompressed.

---

## Core Requirements

Your CLI tool **must**:

### 1. Download & Extract
- Download the ZIP from the URL above (no manual pre-downloading allowed — the tool must fetch it)
- Extract and parse the CSV in-memory or stream it — do not require the user to unzip manually
- Handle the file encoding correctly (UTF-8 with BOM is common in Estonian government data)

### 2. Import to Database
- Store all records in a local database
- The choice of database is yours: SQLite, PostgreSQL, DuckDB, etc. — justify your choice
- The schema must reflect the fields above with appropriate types (e.g. dates as DATE, not TEXT)
- Use `ariregistri_kood` as the primary key

### 3. CLI Interface
The tool must be invokable from the command line with at least these subcommands:

```bash
# Download and import all records
$ ./importer import

# Show how many records are in the DB
$ ./importer stats

# Search by name (partial match)
$ ./importer search --name "Tallinn"

# Look up a company by registry code
$ ./importer get --code 12345678

# Show only active companies
$ ./importer search --status "Registrisse kantud"
```

Feel free to add more commands — creativity is rewarded.

### 4. Progress Feedback
- Show a progress indicator during download and import (progress bar, record count, percentage, etc.)
- Print a summary at the end: total records imported, time elapsed, records per second

---

## Grading Criteria

Your submission will be scored across **five dimensions**, each worth up to 20 points.

---

### ⚡ 1. Import Speed (20 pts)

Raw performance matters. The import pipeline (download → parse → insert) will be benchmarked.

| Time (on reference machine) | Points |
|---|---|
| > 120 seconds | 0 |
| 60–120 seconds | 8 |
| 30–60 seconds | 12 |
| 15–30 seconds | 16 |
| < 15 seconds | 20 |

**Tips that may help:**
- Batch inserts instead of row-by-row
- Streaming the CSV rather than loading it fully into memory
- Disabling indexes during bulk load, rebuilding after
- Using COPY (Postgres) or `.import` (SQLite) style bulk loaders
- Parallelism where it makes sense

---

### ✅ 2. Correctness & Data Integrity (20 pts)

- All ~300k records must be present after import (±0)
- Dates must be stored as proper date types, not strings
- NULL values must be handled correctly (empty fields ≠ `"null"` strings)
- Re-running `import` must not create duplicates (idempotent)
- `ariregistri_kood` must be the primary key and enforced as unique

**Deductions:**
- Wrong record count: −8 pts
- Dates stored as strings: −4 pts
- Duplicate records on re-import: −5 pts
- NULL fields stored as `"null"` strings: −3 pts

---

### 🛡️ 3. Robustness & Error Handling (20 pts)

- What happens if the network drops mid-download?
- What happens if the CSV has a malformed row?
- What if the DB already exists and has data?
- What if the tool is interrupted (Ctrl+C)?

You should handle these gracefully — meaningful error messages, no crashes, ideally resumable where possible.

---

### 🧑‍💻 4. Code Quality (20 pts)

- Is the code readable and well-organised?
- Are functions and modules logically separated?
- Is there a `README.md` explaining how to install and run the tool?
- Are dependencies clearly declared (e.g. `requirements.txt`, `go.mod`, `package.json`)?
- No hardcoded secrets, paths, or magic numbers without explanation

---

### 🌟 5. Bonus Features (20 pts)

Go beyond the basics. Ideas (pick any, or invent your own):

| Feature | Points |
|---|---|
| `--format json\|csv\|table` output flag on search/get commands | +4 |
| Incremental update mode: only import records changed since last run | +6 |
| Export filtered results to CSV | +3 |
| `--county` filter to search by Estonian county (maakond) | +3 |
| Shell autocompletion (bash/zsh) | +2 |
| Docker / `docker-compose` setup so the tool runs in one command | +4 |
| Support for multiple databases via a `--db` flag (e.g. sqlite vs postgres) | +4 |
| Unit or integration tests with >60% coverage | +5 |

Maximum bonus points capped at 20.

---

## Deliverables

Submit a Git repository (GitHub, GitLab, etc.) containing:

```
/
├── README.md          ← Required: setup and usage instructions
├── importer[.py/.go/etc.]  ← Entry point
├── src/ or lib/       ← Source code
├── requirements.txt   ← Or equivalent dependency file
└── (optional) Dockerfile
```

The README must include:
1. How to install dependencies
2. How to run the import
3. Your measured import time on your own machine (hardware specs included)
4. Any design decisions worth noting (DB choice, concurrency model, etc.)

---

## Rules

- You may use any programming language and any open-source libraries
- The tool must run on Linux (Ubuntu 22.04+) and/or macOS
- Do not commit the downloaded data file — the tool must fetch it at runtime
- AI-assisted coding is allowed; however, you must be able to explain every part of your code
- The data is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) — no redistribution of the raw data is required, as the tool downloads it live

---

## Hints & Gotchas

- The CSV uses **semicolons** as delimiters, not commas
- Some fields are regularly empty — plan your schema for NULLs
- `ettevotja_esmakande_kpv` is formatted as `DD.MM.YYYY` — convert it on import
- The file is updated daily; a good solution handles re-running gracefully
- SQLite in WAL mode is significantly faster for bulk writes than the default journal mode

---

## Evaluation Process

1. Reviewer clones your repo on a clean machine
2. Runs the setup steps from your README
3. Runs `./importer import` and measures wall-clock time
4. Runs `./importer stats` to verify record count
5. Runs a series of search/get queries to check correctness
6. Reads source code for quality review
7. Tests error scenarios (kill mid-import, corrupt input, etc.)

Good luck — and may your inserts be fast! 🚀
