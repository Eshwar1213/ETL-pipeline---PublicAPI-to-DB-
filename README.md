# Data Pipeline: Indian Universities ETL

A lightweight, robust Python-based Extract, Transform, Load (ETL) pipeline that automates the ingestion, cleaning, and storage of higher education institutional data. The pipeline fetches live information about Indian academic institutions from a public Web API, filters and processes the datasets using `pandas`, and stores the structured outputs into a local transactional relational database system utilizing `SQLAlchemy`.

---

## 🛠️ Tech Stack & Dependencies

* **Language:** Python 3.13+
* **Data Manipulation:** `pandas`
* **HTTP Ingestion:** `requests`
* **Database Abstraction:** `SQLAlchemy`
* **Storage Core:** SQLite 3

---

## 🔄 Architectural Workflow

The system follows a modular, decoupled ETL design pattern consisting of three core stages:

```text
 [ Public API ] 
       │ (HTTP GET JSON Request)
       ▼
 ┌───────────┐
 │  Extract  │ ──► Fetches 474 raw institutional records
 └───────────┘
       │ (Raw Python Lists/Dicts)
       ▼
 ┌───────────┐
 │ Transform │ ──► RegEx case-insensitive filtering & structural string casting
 └───────────┘
       │ (Cleaned DataFrame - 291 Target Records)
       ▼
 ┌───────────┐
 │   Load    │ ──► Relational DB Engine Schema Mapping
 └───────────┘
       │ (SQLAlchemy ORM Layer)
       ▼
[ universities.db ] ──► Final SQLite binary file (ind_uni table)
```

---

### 1. Extract Stage
* **Mechanism:** Issues an asynchronous-safe programmatic synchronous `HTTP GET` request targeting the target REST endpoint (`http://universities.hipolabs.com/search?country=India`).
* **Payload Ingestion:** Parses incoming binary streams dynamically into native serializable multi-tiered JSON dictionaries.
* **Output Metrics:** Captures baseline metrics, extracting approximately **474 raw entity records** representing multiple classifications of Indian higher educational structures.

### 2. Transform Stage
* **Granular Filtering:** Implements case-insensitive substring searching via Vectorized String Methods in `pandas` (`df['name'].str.contains("university", case=False)`). This isolates true universities, filtering out autonomous engineering institutes, polytechnic schools, or colleges.
* **Data Type Normalization:** The incoming payload nests domain and URL metadata inside multi-valued JSON arrays. This stage sanitizes and flattens arrays into standardized, comma-delimited strings to prevent data duplication and conform to First Normal Form (1NF).
* **Index Re-alignment:** Resets the metadata indices post-drop operations to provide uniform, clean structural alignment ready for downstream relational mapping.
* **Output Metrics:** Drops 183 auxiliary rows, producing **291 highly target-specific university rows**.

### 3. Load Stage
* **DB Abstraction:** Initializes a local dialect database engine context using `create_engine("sqlite:///universities.db")`.
* **Storage Target:** Persists the processed payload directly into a schema-defined table named `ind_uni`.
* **Write Strategy:** Leverages the database persistence policy to seamlessly manage sequential pipeline runs without triggering constraint violations.
