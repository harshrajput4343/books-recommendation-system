## Book Recommender System

A clean, config-first scaffold for building a production-ready Book Recommender System. The repository is set up with a modular pipeline (ingestion → validation → transformation → model training), a local installable package (`books_recommender`), and a place for a Streamlit app. It also ships with the Book-Crossing dataset CSVs for experimentation.

This project is currently a scaffold: core stage files and `app.py` are intentionally empty so you can implement the logic step-by-step. Use this as a starting point for your own recommender.

---

## What’s inside

- Dataset: Book-Crossing CSVs (`notebook/BX-Books.csv`, `BX-Users.csv`, `BX-Book-Ratings.csv`)
- Modular package: `books_recommender/` with folders for components, configuration, entities, logging, exceptions, pipelines, and utilities
- Config file placeholder: `config/config.yaml`
- Streamlit app entrypoint placeholder: `app.py`
- Dockerfile placeholder for containerization
- Notebook for exploration: `notebook/research.ipynb`

---

## Repository structure

```
.
├── app.py                        # Streamlit app entrypoint (empty; implement UI here)
├── Dockerfile                    # Container spec (empty; see Docker section)
├── LICENSE                       # MIT License
├── README.md                     # You are here
├── requirements.txt              # Project dependencies (installs local package with -e .)
├── setup.py                      # Package metadata for books_recommender
├── template.py                   # Script used to generate this scaffold
├── config/
│   └── config.yaml               # Project configuration (empty; see example below)
├── books_recommender/
│   ├── components/               # Pipeline stages (placeholders)
│   │   ├── stage_00_data_ingestion.py
│   │   ├── stage_01_data_validation.py
│   │   ├── stage_02_data_transformation.py
│   │   └── stage_03_model_trainer.py
│   ├── config/
│   │   └── configuration.py      # Reads config.yaml and provides config objects (placeholder)
│   ├── constant/                 # Add project-wide constants
│   ├── entity/
│   │   └── config_entity.py      # Typed config dataclasses (placeholder)
│   ├── exception/
│   │   └── exception_handler.py  # Custom AppException with rich context
│   ├── logger/
│   │   └── log.py                # Centralized logging (placeholder)
│   ├── pipeline/
│   │   └── training_pipeline.py  # Orchestration of stages (placeholder)
│   └── utils/
│       └── util.py               # Utility helpers (placeholder)
└── notebook/
		├── BX-Book-Ratings.csv
		├── BX-Books.csv
		├── BX-Users.csv
		├── research.ipynb            # EDA / prototyping
		└── artifacts/                # Put intermediate outputs here
```

---

## Getting started (Windows PowerShell)

Prerequisites:
- Python 3.8+ recommended (project metadata allows >=3.7)
- Git (optional but recommended)

Create and activate a virtual environment, then install dependencies:

```powershell
# from the repository root
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install -r requirements.txt

# Install the local package in editable mode (already included via -e . in requirements.txt)
# pip install -e .
```

Notes:
- `requirements.txt` includes `streamlit`, `scikit-learn`, `pandas`, `numpy`, `pyYAML`, and `-e .` to install `books_recommender` locally.
- If activation is blocked by policy, run PowerShell as Administrator and execute:
	`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`

---

## Configuration

Project settings live in `config/config.yaml` (currently empty). You can define paths, split params, model hyperparameters, and artifact locations here. A simple starting point:

```yaml
artifacts_root: notebook/artifacts

data_ingestion:
	books_csv: notebook/BX-Books.csv
	users_csv: notebook/BX-Users.csv
	ratings_csv: notebook/BX-Book-Ratings.csv

data_validation:
	required_columns:
		books: [ISBN, Book-Title, Book-Author]
		users: [User-ID]
		ratings: [User-ID, ISBN, Book-Rating]

data_transformation:
	min_user_interactions: 5
	min_item_interactions: 5

model_trainer:
	algorithm: "knn_cosine"     # e.g., user/item-based CF with cosine similarity
	top_k: 50
	seed: 42
```

Implement `books_recommender/config/configuration.py` to parse this YAML and return typed config objects (see `entity/config_entity.py`).

---

## Pipeline (planned)

The pipeline is intentionally modular. Implement each stage in the corresponding file:

1) Data Ingestion (`components/stage_00_data_ingestion.py`)
- Load CSVs from paths in `config.yaml`
- Basic cleaning: types, missing values, deduplication

2) Data Validation (`components/stage_01_data_validation.py`)
- Schema checks: required columns present, value ranges
- Row/column counts, sanity checks

3) Data Transformation (`components/stage_02_data_transformation.py`)
- Build user–item interactions matrix
- Filter users/items by minimum interactions
- Split train/validation (temporal or random)

4) Model Trainer (`components/stage_03_model_trainer.py`)
- Train the recommender (e.g., KNN cosine, matrix factorization, implicit ALS)
- Save model artifacts (embeddings, similarity index, etc.) under `artifacts_root`

Orchestration (`pipeline/training_pipeline.py`)
- Wire the stages together using configuration objects
- Save intermediate artifacts and logs per run

---

## Running the app (placeholder)

`app.py` is reserved for a Streamlit UI. After you implement the pipeline and loadable artifacts, you can expose a simple search + recommend interface. For example:

```python
# app.py (sketch)
import streamlit as st
from books_recommender.utils.util import load_artifacts, recommend_for_title

st.title("Book Recommender")
title = st.text_input("Enter a book title")
if st.button("Recommend") and title:
		model, data = load_artifacts()
		recs = recommend_for_title(title, model, data, top_n=10)
		for r in recs:
				st.write(f"• {r}")
```

Run it:

```powershell
streamlit run app.py
```

---

## Notebooks

Use `notebook/research.ipynb` to explore the Book-Crossing data, test transformations, and prototype models. Move productionized logic into the `books_recommender` package.

---

## Docker (placeholder)

`Dockerfile` exists but is empty. Once you add content, a typical flow is:

```dockerfile
# Example Dockerfile (sketch)
FROM python:3.10-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 8501
CMD ["streamlit", "run", "app.py", "--server.address=0.0.0.0"]
```

Build and run:

```powershell
docker build -t book-recommender .
docker run -p 8501:8501 book-recommender
```

---

## Logging & error handling

- A custom `AppException` is provided in `books_recommender/exception/exception_handler.py`. It enriches errors with file name and line number for easier debugging.
- Implement `books_recommender/logger/log.py` to configure a project-wide logger (e.g., rotating file + console). Use the logger in every stage for traceability.

---

## Development tips

- Keep business logic in `books_recommender/` and treat notebooks as scratch pads.
- Make stages idempotent and save intermediate results to `notebook/artifacts/`.
- Prefer config-driven parameters (avoid hard-coding paths or hyperparameters).
- Add light unit tests as you implement utilities and transformations.

---

## Roadmap

- [ ] Implement configuration loader (`config/configuration.py`)
- [ ] Fill each pipeline stage with working logic
- [ ] Add a persisted model format and artifact loader in `utils/util.py`
- [ ] Implement Streamlit UI in `app.py`
- [ ] Write a functional `Dockerfile` for containerized deployment

---

## License

This project is licensed under the MIT License – see `LICENSE` for details.

---

## Acknowledgements

- Book-Crossing dataset by Cai-Nicolas Ziegler: contains books, users, and explicit ratings.
- The scaffold (`template.py`) auto-generated the folder/file layout used in this repo.

