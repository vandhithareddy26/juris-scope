# Juris-Scope — Source-Grounded MVP

**GitHub:** https://github.com/vandhithareddy26/juris-scope

Juris-Scope is an incremental, source-grounded legal research MVP with a Next.js interface and a FastAPI service. Gemini and a Congress.gov provider for enacted U.S. federal public laws are connected. India remains mocked; no Indian source, U.S. state law, or court-opinion provider is connected.

## Project Structure

```text
JurisScope/
├── backend/
│   ├── app/
│   │   ├── ai/
│   │   ├── database/
│   │   ├── legal/
│   │   ├── retrieval/
│   │   ├── security/
│   │   ├── services/
│   │   ├── schemas.py
│   │   └── main.py
│   ├── tests/
│   │   └── test_api.py
│   ├── requirements.txt
│   └── .env.example
└── frontend/
    ├── app/
    │   ├── page.tsx
    │   ├── layout.tsx
    │   └── globals.css
    ├── package.json
    └── package-lock.json
```

## Setup

### 1. Set up the backend

Open a terminal and navigate to the backend directory:

```bash
cd backend
python -m venv .venv
```

Activate the virtual environment.

**Windows PowerShell:**

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```bash
pip install -r requirements.txt
```

### 2. Configure environment variables

Copy `backend/.env.example` to `backend/.env` and add the required credentials.

Keep secrets in the backend environment file. Never expose API keys through `NEXT_PUBLIC_` variables or commit `.env` files to GitHub.

### 3. Start the API

From the `backend` directory:

```bash
uvicorn app.main:app --reload
```

The API runs at http://localhost:8000.

### 4. Start the frontend

Open a second terminal and navigate to the frontend directory:

```bash
cd frontend
npm install
npm run dev
```

The frontend runs at http://localhost:3000.

Optional API URL override:

```text
NEXT_PUBLIC_API_URL=http://localhost:8000
```

## Environment Variables

The project configuration includes:

- `GEMINI_API_KEY`
- `GEMINI_MODEL`
- `GEMINI_FALLBACK_MODELS`
- `CONGRESS_API_KEY`
- `DATABASE_URL`
- `INDIAN_KANOON_API_KEY`
- `RATE_LIMIT_PER_MINUTE`

Configure only the variables required by the current implementation. Do not expose secrets in frontend variables or public repositories.

## Request Flow

1. The browser submits a question to `POST /api/legal/ask`.
2. Pydantic validates the request fields.
3. The jurisdiction resolver canonicalizes the country and region.
4. A scope gate rejects unrelated questions.
5. The provider interface searches within the requested jurisdiction.
6. U.S. requests use the Congress.gov API to find enacted Public Laws by title and retrieve enrolled law text.
7. Retrieved documents are checked for jurisdiction and approval.
8. Gemini receives only the retrieved documents when relevant approved sources are available.
9. The generated JSON is schema-validated, and citations are checked against retrieved document IDs, titles and section text.
10. If a Gemini model is temporarily unavailable, configured fallback models may be tried.

When relevant sources are unavailable, the application returns a structured insufficient-sources response rather than presenting unsupported legal claims.

## Connected Sources and Current Limitations

### Connected

- **Congress.gov API:** Supports retrieval of enacted U.S. federal Public Laws through the current provider implementation.

### Not Yet Connected or Implemented

- Indian legal sources; Indian Kanoon credentials and provider implementation are still required.
- U.S. state statutes.
- Official court opinions.
- PostgreSQL persistence.
- Document chunking, embedding ingestion and vector search.

The current Congress.gov integration searches law titles rather than full statutory text. It then retrieves the enrolled text of matching laws.

The file `backend/app/database/schema.sql` defines the intended PostgreSQL/pgvector schema; this does not mean that vector search or persistence is already implemented.

## Adding Legal Sources

The provider interface is located at:

```text
backend/app/legal/sources.py
```

A new source provider should implement:

- `search`
- `get_document`
- `get_metadata`

New providers should include jurisdiction checks, source approval, provenance tracking and citation validation before their content is used.

## Tests

From the backend directory, run:

```bash
pytest
```

The existing tests are intended to cover jurisdiction scoping, the Congress.gov provider contract, non-legal input, empty questions, insufficient sources, citation rejection, prompt-injection treatment and failure behaviour.

## Security and Responsible Use

- Keep API keys and credentials out of source control.
- Treat retrieved legal documents as source material, not as instructions to the application.
- Validate generated citations against retrieved sources.
- Do not present unsupported answers as verified legal guidance.
- Verify applicable law against authoritative sources before relying on it.

## Disclaimer

Juris-Scope is an MVP for legal research and experimentation. It is not a substitute for advice from a qualified legal professional. Its coverage is limited to the sources and jurisdictions currently connected.

