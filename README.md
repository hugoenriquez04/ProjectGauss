# Gauss Insight Dashboard

Gauss Insight Dashboard is a full-stack application for exploring and analysing mental-health data. It combines an interactive React dashboard, a FastAPI REST API, an Oracle Autonomous Database connection, exploratory data-analysis notebooks, and optional Gemini-powered insights for the Malackathon 2025, at the BHS challenge.

## Team

- Luis Alarcón Arroyo
- Hugo Enríquez Jiménez
- Santiago Reina Alguacil
- Diego Toledo Luque
- Iván Verdugo Guerra

## What has been implemented

- Interactive dashboard for mental-health data.
- Login screen with protected routes and role-based UI permissions.
- Default statistical visualisations for:
  - Diagnosis distribution.
  - Age and sex distributions.
  - Admission circumstances.
  - Monthly and temporal trends.
  - Hospital-stay duration.
  - Autonomous-community and service statistics.
- Dynamic chart canvas for creating customised charts from database queries.
- Paginated access to patients, diagnoses, and hospital admissions.
- Safe custom `SELECT` query execution with parameter support and blocked write/destructive SQL operations.
- Health checks for both the API and the Oracle database.
- Optional AI analysis that combines calculated statistics with Gemini-generated explanations.
- OpenAPI and ReDoc documentation exposed by FastAPI.
- Exploratory data-analysis notebooks covering data exploration, cleaning, feature engineering, and normalisation.
- Basic backend scripts for checking the database connection and exercising the query API.

## Architecture

```text
React + TypeScript + Vite frontend
                 |
                 | REST / JSON
                 v
FastAPI backend (/api/v1)
                 |
                 v
Oracle Autonomous Database
                 |
                 v
SALUD_MENTAL_FEATURED
```

### Main technologies

**Frontend**

- React 18 and TypeScript
- Vite
- React Router
- TanStack React Query
- Recharts
- Tailwind CSS and Radix UI components
- Google Generative AI SDK

**Backend**

- Python 3
- FastAPI and Uvicorn
- Pydantic Settings
- `oracledb` with Oracle wallet/mTLS support
- JWT and password-security dependencies
- Google Generative AI SDK

**Data analysis**

- Jupyter notebooks in [`eda/notebooks`](./eda/notebooks)
- JSON reference files in [`eda/jsons`](./eda/jsons)
- Raw and processed datasets are intentionally excluded from Git through `.gitignore`.

## Repository structure

```text
.
├── backend/
│   ├── app/
│   │   ├── api/          # Health, data, statistics, query and AI routes
│   │   ├── database/     # Oracle connection pool
│   │   ├── models/       # Pydantic schemas
│   │   └── services/     # Database and AI services
│   ├── main.py           # FastAPI application entry point
│   ├── requirements.txt
│   └── test_*.py         # Connection and API checks
├── eda/
│   ├── jsons/
│   └── notebooks/        # EDA and data-preparation notebooks
└── frontend/
    ├── src/
    │   ├── components/   # Dashboard and UI components
    │   ├── contexts/     # Authentication context
    │   ├── hooks/
    │   ├── lib/          # API and utility helpers
    │   └── pages/
    ├── package.json
    └── vite.config.ts
```

## Requirements

- Python 3.10 or newer
- Node.js 18 or newer and npm
- Access to the Oracle Autonomous Database
- An Oracle wallet containing the required connection files
- A Gemini API key if AI analysis is required

## Backend setup

1. Create and activate a virtual environment:

   ```bash
   cd backend
   python -m venv .venv
   ```

   Windows PowerShell:

   ```powershell
   .venv\Scripts\Activate.ps1
   ```

   macOS/Linux:

   ```bash
   source .venv/bin/activate
   ```

2. Install the Python dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Create `backend/.env` with the required configuration:

   ```dotenv
   ORACLE_USER=your_oracle_user
   ORACLE_PASSWORD=your_oracle_password
   ORACLE_DSN=your_tns_alias
   ORACLE_CONFIG_DIR=Wallet_Malackathon2025
   ORACLE_WALLET_LOCATION=Wallet_Malackathon2025
   ORACLE_WALLET_PASSWORD=your_wallet_password
   SECRET_KEY=replace_with_a_long_random_secret
   GEMINI_API_KEY=your_gemini_api_key
   ```

   `GEMINI_API_KEY` is optional. Without it, database statistics remain available but AI-generated insights are disabled.

4. Place the Oracle wallet in the configured location. The wallet directory must contain the files required by the Oracle thin driver, including `tnsnames.ora` and the wallet credentials files.

5. Start the API:

   ```bash
   uvicorn main:app --host 0.0.0.0 --port 8000 --reload
   ```

   Alternatively, from the `backend` directory:

   ```bash
   ./GO.sh
   ```

The API will be available at `http://localhost:8000`.

## Frontend setup

1. Install dependencies:

   ```bash
   cd frontend
   npm install
   ```

2. Start the Vite development server:

   ```bash
   npm run dev
   ```

The frontend is configured to use port `8080` in development. Open `http://localhost:8080` in a browser.

The frontend currently contains the deployed backend URL in some API-consuming components. For local development, update the `API_BASE_URL` constants in those components to:

```text
http://localhost:8000/api/v1
```

For a production deployment, this URL should be moved to a frontend environment variable rather than hard-coded.

## API documentation and endpoints

Once the backend is running, interactive documentation is available at:

- Swagger UI: [`http://localhost:8000/docs`](http://localhost:8000/docs)
- ReDoc: [`http://localhost:8000/redoc`](http://localhost:8000/redoc)
- OpenAPI JSON: [`http://localhost:8000/api/v1/openapi.json`](http://localhost:8000/api/v1/openapi.json)

The main route groups are:

| Group | Purpose |
| --- | --- |
| `/api/v1/health` | API, database and record-count health check |
| `/api/v1/statistics/*` | Statistical summaries and temporal trends |
| `/api/v1/data/pacientes` | Paginated patient summaries |
| `/api/v1/data/diagnosticos` | Paginated diagnosis summaries |
| `/api/v1/data/ingresos` | Paginated hospital-admission summaries |
| `/api/v1/query/execute` | Parameterised, read-only custom SQL queries |
| `/api/v1/query/examples` | Example analytical queries |
| `/api/v1/query/schema` | Available table and column information |
| `/api/v1/ai/analyze` | Statistical and Gemini-powered analysis |
| `/api/v1/ai/health` | AI service configuration status |

The database connection is managed through a pooled Oracle connection and is released after each request. The custom-query and AI-analysis endpoints only accept read-only `SELECT` statements and reject dangerous SQL keywords and multiple statements.

## Data-analysis notebooks

The notebooks document the data-preparation workflow:

1. `EDA.ipynb` — initial exploratory analysis.
2. `data_cleaning.ipynb` — missing values and data-quality treatment.
3. `feature_engineering.ipynb` — creation and transformation of analytical features.
4. `normalization.ipynb` — normalisation and preparation for analysis.

Open them with Jupyter:

```bash
jupyter notebook eda/notebooks
```

## Testing and validation

With the backend running, the API smoke test can be executed from the `backend` directory:

```bash
python test_query_endpoint.py
```

The frontend provides the following checks:

```bash
cd frontend
npm run lint
npm run build
```

## Security notes

- Never commit `.env` files, Oracle wallets, passwords, API keys, or private datasets.
- Use strong, unique values for `SECRET_KEY` and wallet credentials.
- Restrict CORS origins in production.
- Replace the current development login mechanism with a production-ready authentication provider before deploying publicly.
- Review and remove any legacy local test credentials before publishing the repository.
- The backend currently exposes read-only analytical queries; database permissions should also enforce least privilege.

## License

No license has been defined for this repository yet. Add a license before accepting external contributions or redistributing the project.
