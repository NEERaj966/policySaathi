# PolicySaathi

> **Understand government schemes, policies, and citizen rights in simple language—with evidence you can verify.**

PolicySaathi is a multilingual web application that helps citizens understand official government schemes, policy notifications, legislation, eligibility conditions, required documents, deadlines, and practical next steps.

The project is designed around one core principle:

**Every important answer must be traceable to a reliable source.** If the system cannot find sufficient evidence, it must say so instead of guessing.

---

## 1. Project goals

PolicySaathi aims to:

- Explain complex government documents in simple language.
- Answer questions in Hindi and English first, with an architecture that can support regional languages.
- Find relevant government schemes using a citizen's optional profile information.
- Explain eligibility conditions and identify missing information.
- Highlight rights, duties, deadlines, benefits, and practical effects of policy changes.
- Show official source links and document/page references for factual answers.
- Guide users through application steps without pretending to submit an application on their behalf.

### Initial target users

All citizens, including students, workers, families, small-business owners, and people who find official policy documents difficult to understand.

### Initial platform and language scope

- Platform: responsive web application.
- Initial languages: Hindi and English.
- Future languages: add regional languages after the retrieval, translation, and evaluation pipeline is reliable.

---

## 2. What we are building first

Build a working, evidence-backed MVP before adding advanced features.

### MVP features

1. **Document library**
   - Store official documents and their metadata.
   - Record source organisation, official URL, publication date, document type, language, and verification status.
   - Extract text from PDFs.
   - Keep page numbers and section headings where possible.

2. **Document explanation**
   - Summarise a document in plain language.
   - Extract benefits, eligibility, required documents, dates, duties, and application steps.
   - Allow users to ask questions about a selected document.

3. **Grounded AI assistant (RAG)**
   - Retrieve relevant passages from ingested documents.
   - Answer using retrieved evidence.
   - Attach source links and page/section references.
   - Say “I could not verify this from the available sources” when evidence is insufficient.

4. **Scheme discovery**
   - Search schemes by keyword, state, category, and target group.
   - Display the official source and last-verified date.
   - Do not invent schemes or application URLs.

5. **Eligibility checker**
   - Use explicit, structured rules for conditions that can be evaluated.
   - Return `POTENTIALLY_ELIGIBLE`, `NOT_ELIGIBLE`, or `NEEDS_MORE_INFORMATION`.
   - Explain which conditions passed, failed, or are unknown.
   - Clearly state that this is guidance, not an official government decision.

6. **Hindi and English**
   - Detect or allow selection of the response language.
   - Preserve names, dates, amounts, legal terms, and source references accurately.
   - Never translate a legal condition in a way that changes its meaning.

### Later features (do not block the MVP)

- Voice input and spoken answers.
- More Indian languages.
- Policy-version comparison and change alerts.
- Advanced citizen-rights explorer.
- Saved schemes and reminders.
- Admin review dashboard and scheduled source monitoring.

---

## 3. Recommended technology stack

Use this stack unless the team records a clear technical reason to change it.

| Layer | Technology | Responsibility |
|---|---|---|
| Frontend | React + Vite | User interface |
| Styling | Tailwind CSS | Responsive design |
| Backend | Python + FastAPI | REST API and application logic |
| Main database | MongoDB | Users, documents, schemes, conversations, metadata |
| Vector database | ChromaDB | Embeddings and semantic retrieval |
| Embeddings | Sentence Transformers | Convert text into vectors |
| LLM | Gemini API (or another approved provider) | Explain retrieved evidence |
| PDF extraction | PyMuPDF | Extract text and page information |
| OCR | Tesseract (only when required) | Read scanned PDFs |
| Authentication | JWT in HttpOnly cookies | Authenticated sessions |
| Password hashing | Argon2id or bcrypt | Store password hashes, never plaintext |
| Deployment | Vercel (frontend) + Render (backend) | Public prototype |

**Important:** A model API is not the source of truth. Official documents and verified datasets are the source of truth. The LLM only helps interpret retrieved evidence.

---

## 4. High-level architecture

```text
Browser (React)
    |
    | HTTPS / JSON
    v
FastAPI backend
    |
    +--> Authentication and authorisation
    |
    +--> Document service
    |       |
    |       +--> Validate upload
    |       +--> Extract PDF text / OCR
    |       +--> Split into page-aware chunks
    |       +--> Save metadata in MongoDB
    |       +--> Save embeddings in ChromaDB
    |
    +--> AI / RAG service
    |       |
    |       +--> Detect language and intent
    |       +--> Retrieve relevant chunks
    |       +--> Filter by source and document status
    |       +--> Generate answer from evidence
    |       +--> Validate citations and uncertainty
    |
    +--> Scheme service
    |       |
    |       +--> Search verified schemes
    |       +--> Evaluate structured eligibility rules
    |       +--> Return explanation and official links
    |
    +--> MongoDB
    |
    +--> ChromaDB
    |
    +--> Approved LLM API
```

Start with a **modular monolith**: one FastAPI application with separate modules. Do not create microservices until there is a measured need.

---

## 5. Suggested repository structure

Create this structure gradually. Do not create empty files for every future feature on day one.

```text
policysaathi/
├── README.md
├── README_RULES_AND_SECURITY.md
├── PROGRESS.md
├── .gitignore
├── .env.example
├── frontend/
│   ├── package.json
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       ├── api/
│       ├── components/
│       ├── pages/
│       ├── hooks/
│       ├── context/
│       ├── routes/
│       ├── utils/
│       └── styles/
├── backend/
│   ├── requirements.txt
│   ├── app/
│   │   ├── main.py
│   │   ├── core/
│   │   │   ├── config.py
│   │   │   ├── security.py
│   │   │   └── database.py
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── router.py
│   │   │       └── endpoints/
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── services/
│   │   │   ├── document_service.py
│   │   │   ├── retrieval_service.py
│   │   │   ├── answer_service.py
│   │   │   └── eligibility_service.py
│   │   ├── repositories/
│   │   ├── utils/
│   │   └── tests/
│   └── uploads/              # local development only; never commit uploads
├── data/
│   ├── source_registry/
│   └── samples/              # only permitted sample documents
├── scripts/
│   ├── ingest_document.py
│   └── check_environment.py
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DATABASE.md
│   └── API.md
└── tests/
```

### What belongs where?

- `frontend/src/pages/`: full pages such as Dashboard, Assistant, Schemes, Document Explorer.
- `frontend/src/components/`: reusable UI such as Navbar, SourceCitation, SchemeCard.
- `frontend/src/api/`: functions that call backend endpoints.
- `backend/app/api/`: HTTP route definitions only; keep business logic out of route handlers.
- `backend/app/services/`: actual application logic.
- `backend/app/models/`: MongoDB document models or database access definitions.
- `backend/app/schemas/`: Pydantic request and response validation.
- `backend/app/core/`: configuration, database connection, security utilities.
- `backend/app/repositories/`: database queries.
- `backend/app/tests/`: backend tests.
- `data/source_registry/`: curated metadata about official sources, not private user files.
- `scripts/`: repeatable development and ingestion commands.
- `docs/`: architecture, database, API and team decisions.

---

## 6. Start the project: setup instructions

### Prerequisites

Install:

- Git
- Node.js LTS and npm
- Python 3.11 or newer
- MongoDB (local instance or managed MongoDB Atlas)
- A ChromaDB-compatible setup
- An API key for the selected LLM provider

Check versions:

```bash
git --version
node --version
npm --version
python --version
```

### Step 1 — Create the repository

```bash
mkdir policysaathi
cd policysaathi
git init
```

Create the root documentation files:

```text
README.md
README_RULES_AND_SECURITY.md
PROGRESS.md
.gitignore
.env.example
```

### Step 2 — Create the frontend

```bash
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm install axios react-router-dom lucide-react
cd ..
```

Add Tailwind using the installation instructions for the Tailwind version you select. Do not mix setup instructions from different major versions.

Run the frontend:

```bash
cd frontend
npm run dev
```

### Step 3 — Create the backend

```bash
mkdir backend
cd backend
python -m venv .venv
```

Activate the environment:

**Windows PowerShell**
```powershell
.venv\Scripts\Activate.ps1
```

**Linux / macOS**
```bash
source .venv/bin/activate
```

Install initial dependencies:

```bash
pip install fastapi uvicorn[standard] pymongo motor pydantic-settings python-multipart
pip install pymupdf chromadb sentence-transformers
pip install argon2-cffi python-jose[cryptography] passlib
pip freeze > requirements.txt
```

Create the initial application:

```text
backend/app/main.py
backend/app/core/config.py
backend/app/core/database.py
backend/app/api/v1/router.py
```

Run the backend from the `backend` directory:

```bash
uvicorn app.main:app --reload
```

Open:

- API: `http://127.0.0.1:8000`
- Swagger docs: `http://127.0.0.1:8000/docs`

### Step 4 — Configure environment variables

Create `.env` locally. Use `.env.example` as a template.

Example `.env.example`:

```dotenv
APP_ENV=development
APP_NAME=PolicySaathi
API_V1_PREFIX=/api/v1
FRONTEND_ORIGIN=http://localhost:5173

MONGODB_URI=mongodb://localhost:27017
MONGODB_DATABASE=policysaathi

CHROMA_HOST=localhost
CHROMA_PORT=8001

LLM_PROVIDER=gemini
GEMINI_API_KEY=
LLM_MODEL=

JWT_SECRET=
JWT_ALGORITHM=HS256
ACCESS_TOKEN_MINUTES=15
REFRESH_TOKEN_DAYS=7

COOKIE_SECURE=false
COOKIE_SAMESITE=lax
MAX_UPLOAD_MB=10
```

Generate a strong random secret locally. Never commit `.env`, API keys, passwords, or production secrets.

### Step 5 — Connect services one at a time

Recommended order:

1. Start MongoDB and verify the connection.
2. Start FastAPI and verify `/health`.
3. Start React and verify it can call the backend.
4. Add one official PDF and test extraction.
5. Store extracted page-aware chunks.
6. Add embeddings and retrieval.
7. Add LLM answer generation with citations.
8. Add scheme search.
9. Add rule-based eligibility.
10. Add Hindi response support and test it.

Do not integrate every service at once. Verify each step before moving forward.

---

## 7. Data model overview

Use MongoDB collections for application data. Store vector embeddings in ChromaDB, with stable IDs linking them to MongoDB records.

| Collection | Purpose | Important references |
|---|---|---|
| `users` | Login and account data | `_id` |
| `citizen_profiles` | Optional citizen preferences/attributes | `userId -> users._id` |
| `documents` | Official document metadata and processing status | `_id`, source metadata |
| `document_chunks` | Text chunks with page/section metadata, if stored in MongoDB | `documentId -> documents._id` |
| `schemes` | Structured scheme information | `_id` |
| `eligibility_rules` | Machine-readable eligibility conditions | `schemeId -> schemes._id` |
| `conversations` | Chat sessions | `userId -> users._id` |
| `messages` | Conversation messages | `conversationId -> conversations._id` |
| `saved_schemes` | Schemes saved by users | `userId`, `schemeId` |
| `source_registry` | Approved source organisations/domains | source metadata |
| `document_versions` | Version history and change tracking | `documentId -> documents._id` |

### Understand the relationships

- One user can have zero or one citizen profile.
- One user can have many conversations.
- One conversation can have many messages.
- One document can have many chunks.
- One scheme can have many eligibility rules.
- Users and schemes have a many-to-many relationship, represented by `saved_schemes`.
- A scheme may be supported by multiple official documents, and a document may describe multiple schemes. If this relationship is needed, use a join collection such as `scheme_sources`.

Use MongoDB ObjectIds or another consistent ID type. Keep reference field names consistent across schemas and API responses.

---

## 8. API plan

Use `/api/v1` as the API prefix.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Service health |
| POST | `/api/v1/auth/register` | Create account |
| POST | `/api/v1/auth/login` | Login |
| POST | `/api/v1/auth/logout` | Logout |
| GET | `/api/v1/auth/me` | Current user |
| GET | `/api/v1/schemes` | Search schemes |
| GET | `/api/v1/schemes/{scheme_id}` | Scheme details |
| POST | `/api/v1/schemes/{scheme_id}/eligibility-check` | Evaluate eligibility |
| POST | `/api/v1/documents` | Upload document |
| GET | `/api/v1/documents/{document_id}` | Document metadata |
| POST | `/api/v1/documents/{document_id}/process` | Process document |
| POST | `/api/v1/assistant/chat` | Ask a grounded question |
| POST | `/api/v1/documents/{document_id}/explain` | Explain a document |

Define request/response schemas before implementing each endpoint. Do not return database objects or secrets directly.

---

## 9. Official information sources

Prefer primary government sources. Record the exact page/document URL and the date checked.

Potential starting points:

- MyScheme: https://www.myscheme.gov.in/
- India Code: https://www.indiacode.nic.in/
- Open Government Data Platform: https://www.data.gov.in/
- Press Information Bureau: https://pib.gov.in/
- National Portal of India: https://www.india.gov.in/

Before ingesting a source, check its terms, API documentation, reuse licence, robots/access rules, and whether automated access is permitted. Do not assume a website has a public API just because it has a search page.

For each ingested item, store:

- Official title
- Issuing organisation
- Canonical official URL
- Publication/effective date, if available
- Date retrieved
- Language
- State/region and jurisdiction
- Document type
- Version/status
- Verification status
- Content hash
- Page/section information

Never label an unverified source as verified.

---

## 10. RAG answer format

A useful answer should contain:

1. **Direct answer** in the selected language.
2. **Simple explanation** of the relevant condition or policy.
3. **What this means for the citizen**.
4. **What information is missing**, if any.
5. **Sources** with official title, link, page/section, and date checked.
6. **Uncertainty note** when evidence is incomplete, outdated, conflicting, or jurisdiction-specific.

The model must not create citations. Citations must be assembled from retrieved source metadata, and every cited passage must actually support the associated claim.

---

## 11. Eligibility engine

Do not let the LLM make the final eligibility calculation.

Represent each condition as structured data, for example:

```json
{
  "schemeId": "SCHEME_ID",
  "conditions": [
    {
      "field": "state",
      "operator": "equals",
      "value": "Rajasthan",
      "sourceDocumentId": "DOCUMENT_ID"
    },
    {
      "field": "annualFamilyIncome",
      "operator": "less_than_or_equal",
      "value": 200000,
      "sourceDocumentId": "DOCUMENT_ID"
    }
  ]
}
```

This is an illustrative schema only. Actual values and operators must come from the current official scheme rules.

The engine should distinguish:

- Passed condition
- Failed condition
- Unknown / missing information
- Rule not machine-readable

Return a cautious result and explain the evidence. Do not claim that the government has approved or rejected the citizen.

---

## 12. Development milestones

Use `PROGRESS.md` as the source of truth for task status.

Suggested milestone order:

1. Repository, environment, frontend and backend skeleton.
2. Database connection and health endpoint.
3. Official source registry and document metadata.
4. PDF extraction with page tracking.
5. Chunking and embeddings.
6. Retrieval and evidence-backed answer generation.
7. Citation validation and “insufficient evidence” behaviour.
8. Scheme search and structured data.
9. Rule-based eligibility.
10. Hindi/English UI and answer evaluation.
11. Authentication, security review, and privacy checks.
12. Deployment, end-to-end testing, and demo preparation.

---

## 13. Testing and quality gates

Before calling a feature complete, verify:

- Happy path works.
- Invalid inputs are rejected.
- Empty search results are handled.
- API/network failure is shown clearly.
- No secrets appear in logs or responses.
- A user cannot access another user's private data.
- Citations resolve to the correct official document and page.
- Outdated or unverified sources are clearly labelled.
- Hindi and English answers preserve dates, amounts, names, and conditions.
- Eligibility output explains unknown fields instead of assuming values.

Create tests for retrieval quality, citation correctness, eligibility rules, access control, file validation, and common failure modes.

---

## 14. Team workflow

For a four-person team, a practical split is:

- **Frontend:** React pages, reusable components, API integration, accessibility.
- **Backend:** FastAPI, database, authentication, API schemas, deployment.
- **AI/RAG:** extraction, chunking, embeddings, retrieval, answer generation, citations.
- **Data and quality:** official sources, scheme/rule structuring, verification, test cases, demo validation.

Agree on API contracts and data schemas before parallel implementation. Use small pull requests and review each other's security-sensitive code.

---

## 15. Definition of done

The MVP is ready for demonstration when a user can:

1. Open the deployed web application.
2. Search or select a real, verified official document/scheme.
3. Ask a question in Hindi or English.
4. Receive a useful answer grounded in retrieved official text.
5. Open the cited source and locate the supporting page/section.
6. Check eligibility using explicit rules and see passed/failed/missing conditions.
7. Receive a clear warning when information is insufficient or cannot be verified.

**Do not claim a feature is complete because its button or screen exists. It is complete only when its real backend behaviour has been tested.**

---

## 16. Important project boundaries

PolicySaathi is an information and guidance tool. It is not:

- A government department or official government portal.
- A substitute for an official eligibility decision.
- A lawyer or provider of formal legal advice.
- A service that submits applications unless a separately authorised, secure integration is built and approved.

Display a clear disclaimer and direct users to official channels for final decisions and applications.

---

## 17. First development session checklist

- [ ] Create the Git repository.
- [ ] Add the root documentation files.
- [ ] Create the frontend using Vite.
- [ ] Create the FastAPI backend and virtual environment.
- [ ] Add `.gitignore` and `.env.example`.
- [ ] Make `/health` return a successful response.
- [ ] Make the React frontend call `/health`.
- [ ] Connect MongoDB.
- [ ] Record the completed work in `PROGRESS.md`.

Start small, verify each layer, and keep the first end-to-end path working before adding more features.
