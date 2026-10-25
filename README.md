# Olist Marketplace Intelligence Warehouse

A dbt-powered warehouse on the public Olist e-commerce dataset, built to improve marketplace revenue, delivery reliability, and seller quality.

See the full [project brief](docs/project_brief.md).

## Architecture

```mermaid
flowchart LR
    A[Olist CSV files<br/>Kaggle] --> B[Raw layer]
    B --> C[Staging models<br/>dbt]
    C --> D[Core models<br/>facts + dimensions]
    D --> E[Marts<br/>revenue, SLA, sellers, RFM]
    E --> F[Dashboard]
    G[dbt tests + docs + CI] -.-> C
    G -.-> D
    G -.-> E
```

## Setup (Windows)

Requirements: Python 3.12, Git, DuckDB CLI, Docker Desktop, VS Code.

```
git clone https://github.com/viniska13/olist-marketplace-intelligence-warehouse.git
cd olist-marketplace-intelligence-warehouse
py -3.12 -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
dbt --version
```

## Dashboard screenshots

_Coming soon._