# PolicySaathi (पॉलिसी साथी) — Project Master Guide & Progress Dashboard

> **Understand government schemes, policies, and citizen rights in simple language — grounded in verifiable evidence.**

---

## 📌 Document Overview & Purpose

This master `README.md` acts as the **central cognitive blueprint and progress tracking log** for PolicySaathi. It captures:
1. **What PolicySaathi is** and its core principles.
2. **What we understand** about the architecture, security constraints, and data flows.
3. **What we do and why we do it** (design rationale and architectural guardrails).
4. **Current development progress & status** (honest tracking of what is done, tested, and pending).
5. **Phase-by-phase execution roadmap** for bringing the MVP to production.

---

## 1. Project Vision & Core Understanding

### 1.1 The Problem
Government policies, notifications, legal acts, and welfare schemes in India are often written in dense administrative legalese and scattered across hundreds of disparate portals. Citizens — particularly students, daily wage earners, rural families, and small business owners — struggle to discover what they are entitled to, whether they qualify, what documents they need, and how to apply. Moreover, existing generic AI tools frequently **hallucinate** schemes, invent deadlines, state false eligibility criteria, or produce non-existent government links.

### 1.2 The PolicySaathi Solution
PolicySaathi is an evidence-backed, bilingual (Hindi & English) civic technology platform designed to make government welfare and rights transparent and accessible.

**The Golden Non-Negotiable Rule:**
> **Every factual claim must be directly traceable to an official source document.**  
> If evidence is missing, conflicting, or incomplete, the system must explicitly declare uncertainty instead of guessing.

### 1.3 Target Audience & Scope
- **Target Audience:** All Indian citizens (students, workers, farmers, families, senior citizens, MSMEs).
- **MVP Languages:** Hindi (हिन्दी) and English.
- **Form Factor:** Clean, responsive, accessible web application.
- **Boundaries:** PolicySaathi is an **information and guidance tool**. It is **not** a government portal, **not** a provider of formal legal advice, and **does not** autonomously submit applications on behalf of citizens during the MVP.

---

## 2. Core Pillars & Architecture

### 2.1 High-Level Architecture Diagram

```text
       ┌────────────────────────────────────────────────────────┐
       │                Citizen Web Client (React)              │
       │       (Vite + Tailwind CSS + Lucide Icons + i18n)      │
       └───────────────────────────┬────────────────────────────┘
                                   │ HTTPS / JSON (Cookie Auth)
                                   ▼
       ┌────────────────────────────────────────────────────────┐
       │               FastAPI Modular Monolith                 │
       │                                                        │
       │  ┌──────────────────┐  ┌─────────────────────────────┐ │
       │  │ Auth & Security  │  │   Document Pipeline         │ │
       │  │ (Argon2id, JWT,  │  │   (PyMuPDF, Chunking,       │ │
       │  │  HttpOnly cookie)│  │    OCR fallback, Hashes)    │ │
       │  └──────────────────┘  └──────────────┬──────────────┘ │
       │  ┌──────────────────┐                 ▼                │
       │  │ Scheme Search &  │  ┌─────────────────────────────┐ │
       │  │ Rule-based Engine│  │   Grounded RAG Assistant    │ │
       │  │ (Deterministic)  │  │   (Passage Retrieval,       │ │
       │  │                  │  │    Citation Validation,     │ │
       │  └──────────────────┘  │    Gemini LLM Synthesis)    │ │
       │                        └──────────────┬──────────────┘ │
       └───────────────────────────────┬───────┴────────────────┘
                                       │
            ┌──────────────────────────┴──────────────────────────┐
            ▼                                                     ▼
┌───────────────────────────────┐             ┌───────────────────────────────┐
│        MongoDB Database       │             │       ChromaDB Vector DB      │
│  - users & citizen_profiles   │             │  - Chunk embeddings           │
│  - documents & chunks metadata│             │  - Document & section metadata│
│  - schemes & eligibility_rules│             │  - Sentence Transformers      │
│  - conversations & messages   │             └───────────────────────────────┘
│  - source_registry            │
└───────────────────────────────┘
```

### 2.2 Technology Stack

| Layer | Chosen Technology | Justification ("Why this tech?") |
|---|---|---|
| **Frontend** | React (Vite) + Tailwind CSS | High performance, modular component architecture, rapid UI styling for responsive mobile/desktop interfaces. |
| **Backend** | Python 3.11+ & FastAPI | Asynchronous I/O, auto-generated OpenAPI/Swagger documentation, native Pydantic data validation, strong AI/ML ecosystem. |
| **Primary Database** | MongoDB (via Motor/PyMongo) | Flexible document store for evolving policy schemas, nested eligibility conditions, multilingual fields, and metadata. |
| **Vector Store** | ChromaDB | Lightweight, open-source embedding store with strong metadata filtering (by document ID, jurisdiction, date). |
| **Embeddings** | Sentence Transformers (`all-MiniLM-L6-v2` or multilingual) | Fast, local vector representations without per-token external API costs or data leakage. |
| **LLM Provider** | Google Gemini API | High-context reasoning, strong multilingual translation and explanation capabilities, strictly grounded on retrieved passages. |
| **Document Parser** | PyMuPDF (fitz) + Tesseract OCR | Accurate text extraction with page/bounding-box tracking; OCR fallback for scanned gazette notifications. |
| **Security & Auth** | Argon2id + Secure HttpOnly Cookies | Resistance to GPU cracking, zero exposure of session tokens to client-side JavaScript (`localStorage`). |

---

## 3. "What We Do and Why We Do It" (Design Rationale & Decisions)

To avoid common AI development pitfalls, the project strictly adheres to these deliberate decisions:

### 1. Why a Modular Monolith instead of Microservices?
- **What we do:** Build a single FastAPI codebase cleanly organized into `api/`, `services/`, `models/`, and `repositories/`.
- **Why we do it:** Microservices introduce distributed transaction overhead, network latency, Docker/Kubernetes complexity, and deployment fragility. For an MVP, a modular monolith provides identical separation of concerns with vastly easier local development, testing, and deployment.

### 2. Why Deterministic Eligibility instead of LLM Calculations?
- **What we do:** Implement a pure Python rule-evaluator that compares structured user attributes against scheme rules (`field`, `operator`, `value`). The engine returns `POTENTIALLY_ELIGIBLE`, `NOT_ELIGIBLE`, or `NEEDS_MORE_INFORMATION`.
- **Why we do it:** Large Language Models are probabilistic and cannot be trusted for deterministic math or threshold evaluations (e.g., income limits, age caps, cutoff dates). The LLM is only permitted to *explain* why a rule passed or failed, never to compute it.

### 3. Why Strict Retrieval-Grounded Citations?
- **What we do:** Pass only verified chunk passages to the LLM prompt. Citations (URL, page number, document title, verified date) are built by backend code from retrieved chunk metadata, not invented by the LLM.
- **Why we do it:** Prevents hallucination. If the model is asked a question not covered by retrieved chunks, it is instructed to respond with "Insufficient evidence" rather than fabricating an answer.

### 4. Why Page-Aware Chunking?
- **What we do:** When ingesting PDFs with PyMuPDF, preserve exact page numbers and section headers in both MongoDB chunk records and ChromaDB metadata.
- **Why we do it:** Citizen trust requires verification. An answer stating "Available up to ₹50,000 subsidy (Source: PM SVANidhi Guidelines, Page 4, Section 3.2)" allows a citizen or official to verify the fact in seconds.

### 5. Why Privacy-Preserving Profiles?
- **What we do:** Citizen profiles are strictly optional. We explicitly refuse to collect Aadhaar numbers, PAN, bank account numbers, passwords, OTPs, or unnecessary caste/health data.
- **Why we do it:** Data minimization is critical. Scheme discovery does not require identity verification. Minimizing personal data protects citizens and ensures compliance with privacy best practices.

---

## 4. Non-Negotiable Engineering & Security Rules

1. **No Fake Data as Real:** Demo data must be explicitly labeled as sample data.
2. **No Fake or Model-Generated Citations:** Every cited link/page must exist in the database.
3. **No Hardcoded Secrets:** All secrets, keys, and tokens live strictly in `.env`.
4. **No Frontend-Only Security:** Permissions and ownership checks must execute on the backend (`current_user.id == resource.user_id`).
5. **No Blind Web Scraping:** Only approved, verified government domains in `source_registry`.
6. **No Autonomous Submissions:** The MVP strictly informs and guides; it does not execute transactions on government portals.
7. **Strict File Upload Controls:** Max 10MB, PDF magic-byte validation, server-side UUID file naming, outside public web root, isolated processing.
8. **No Premature Optimization / Scope Creep:** No mobile apps, blockchain, Kubernetes, or voice features until text-based MVP is rock solid.

---

## 5. Master Progress Tracker

### Overall Status Summary
- **Current Phase:** Phase 1 — Repository & Foundation Setup
- **Overall Completion:** 10%
- **Status Date:** October 2026
- **Current Sprint Goal:** Establish clean repository root, initialize frontend (React Vite) & backend (FastAPI), verify MongoDB & ChromaDB connectivity, and implement `/health` check.

### Status Legend
- `[ ]` Not started
- `[-]` In progress
- `[x]` Completed and verified with tests
- `[!]` Blocked
- `[~]` Deferred / Future Milestone

---

### Phase Checklist

#### Phase 1: Planning, Standards & Repository Foundation
- [x] Review project starter specifications and security rules.
- [x] Read and cross-verify `README.md`, `README_RULES_AND_SECURITY.md`, `PROGRESS.md`, `ARCHITECTURE.md`, `DATABASE.md`.
- [x] Create project root `README.md` tracking architecture, decisions, and progress dashboard.
- [ ] Initialize Git repository (`git init`) and commit documentation foundation.
- [ ] Setup root `.gitignore` and `.env.example`.
- [ ] Define shared API contracts and data schemas.

#### Phase 2: Environment & Scaffolding
- [ ] Verify local Python 3.11+ and Node.js LTS installation.
- [ ] Scaffold React + Vite frontend (`npm create vite@latest frontend -- --template react`).
- [ ] Configure Tailwind CSS, Axios, and Lucide Icons in frontend.
- [ ] Create Python virtual environment (`backend/.venv`) and install core dependencies (`fastapi`, `uvicorn`, `pymongo`, `motor`, `pydantic-settings`, `chromadb`, `pymupdf`).
- [ ] Verify local MongoDB connection (`mongodb://localhost:27017`).
- [ ] Configure ChromaDB instance.
- [ ] Verify `.env` file security (ensuring it is untracked by Git).

#### Phase 3: Backend Core & Health Check
- [ ] Create FastAPI entry point (`backend/app/main.py`).
- [ ] Implement settings loader (`backend/app/core/config.py`).
- [ ] Configure MongoDB async client with lifecycle events (`backend/app/core/database.py`).
- [ ] Implement API v1 router (`backend/app/api/v1/router.py`).
- [ ] Add `/health` endpoint returning system and database status.
- [ ] Implement global exception handler and standardized error response.
- [ ] Configure structured logging with secret redaction.

#### Phase 4: Frontend Foundation & Health Connection
- [ ] Setup React Router (`/`, `/assistant`, `/schemes`, `/documents`).
- [ ] Create API client service (`frontend/src/api/client.js`).
- [ ] Connect frontend to backend `/health` endpoint to verify end-to-end communication.
- [ ] Create reusable UI layout (Navbar, Footer, Language Switcher, Alert banner).
- [ ] Implement loading, error, and empty state components.

#### Phase 5: Official Source Registry & PDF Ingestion Pipeline
- [ ] Create `source_registry` collection and seed with initial official portals (e.g. MyScheme, IndiaCode).
- [ ] Implement document upload validation (size limit, PDF magic bytes, path traversal defense).
- [ ] Implement PyMuPDF extractor capturing page-by-page text.
- [ ] Add clean text chunking with preserved metadata (page, section, document hash).
- [ ] Implement duplicate detection using SHA-256 document content hash.
- [ ] Store chunk metadata in MongoDB and generate embeddings for ChromaDB.

#### Phase 6: Grounded RAG & Bilingual Assistant
- [ ] Configure embedding pipeline using Sentence Transformers.
- [ ] Build ChromaDB retrieval service with metadata filters (source status, jurisdiction).
- [ ] Integrate Gemini API with strict system instructions (grounded strictly in context).
- [ ] Implement citation generator matching answer claims with retrieved page references.
- [ ] Implement "Insufficient Evidence" fallback mechanism.
- [ ] Test Hindi and English question answering accuracy.

#### Phase 7: Scheme Catalog & Deterministic Eligibility Engine
- [ ] Define `schemes` and `eligibility_rules` MongoDB schemas.
- [ ] Seed verified government scheme records (e.g. PM SVANidhi, PM Kisan).
- [ ] Build scheme search and filter endpoint (by state, category, target demographic).
- [ ] Implement pure Python deterministic rule evaluation engine.
- [ ] Return structured evaluation (`passed`, `failed`, `unknown` conditions).
- [ ] Add citizen-facing eligibility disclaimer.

#### Phase 8: Authentication, Authorization & Privacy
- [ ] Implement registration and login using Argon2id password hashing.
- [ ] Configure HttpOnly, Secure, SameSite cookies for JWT session tokens.
- [ ] Implement user ownership verification middleware (prevent IDOR attacks).
- [ ] Add rate-limiting to auth, search, and assistant endpoints.
- [ ] Build profile management with optional fields only.

#### Phase 9: Testing, Hardening & Deployment
- [ ] Backend unit and integration test suite (`pytest`).
- [ ] Retrieval relevance and hallucination benchmark suite.
- [ ] Security checks: CORS origin whitelist, headers, secret audit.
- [ ] Prepare deployment configuration (Frontend: Vercel; Backend: Render/Container).
- [ ] Create live demo walkthrough scenario.

---

## 6. Sprint Log & Activity History

| Date | Action / Milestone | Rationale ("Why") | Status |
|---|---|---|---|
| **2026-10-30** | Comprehensive review of all starter documentation (`README.md`, `README_RULES_AND_SECURITY.md`, `PROGRESS.md`, `ARCHITECTURE.md`, `DATABASE.md`). | Understand requirements, security boundaries, and architectural guidelines before writing any code. | Completed |
| **2026-10-30** | Created root `README.md` master cognitive dashboard and progress tracker. | Provide a single source of truth for project vision, design decisions ("what and why"), and live status tracking. | Completed |
| **Next Up** | Initialize Git repository, create project directory structure (`frontend/`, `backend/`, `data/`, `docs/`), and configure `.gitignore` & `.env.example`. | Establish clean version control and environment isolation before code scaffolding. | Ready to start |

---

## 7. Immediate Next Steps

To begin implementation cleanly according to the defined specifications:
1. Initialize Git in the project root (`git init`).
2. Set up root `.gitignore` and `.env.example`.
3. Scaffold `backend/` with FastAPI, virtual environment, and `/health` route.
4. Scaffold `frontend/` with Vite React, Tailwind CSS, and health check integration.
5. Verify communication between Frontend and Backend.

