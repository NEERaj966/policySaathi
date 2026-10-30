# PolicySaathi — Project Progress Tracker

> Update this file whenever work is started, completed, blocked, or changed. Keep it honest: a screen is not “done” until its real behaviour is tested.

## Project status

- **Current phase:** Not started
- **Overall completion:** 0% (update as work is completed)
- **Last updated:** YYYY-MM-DD
- **Current sprint owner:** Add name
- **Demo deployment:** Not deployed
- **Known blockers:** None recorded

## Status legend

- `[ ]` Not started
- `[-]` In progress
- `[x]` Completed and tested
- `[!]` Blocked
- `[~]` Deferred / out of MVP scope

---

## 1. Planning and repository

- [ ] Confirm MVP scope and acceptance criteria.
- [ ] Create Git repository.
- [ ] Add `README.md`.
- [ ] Add `README_RULES_AND_SECURITY.md`.
- [ ] Add `.gitignore`.
- [ ] Add `.env.example`.
- [ ] Create project folder structure.
- [ ] Agree on branch and pull-request workflow.
- [ ] Record API contracts and database decisions.

**Completion evidence / notes:**
- Add commit hash, PR, screenshot, or test result here.

## 2. Development environment

- [ ] Install and verify Node.js/npm.
- [ ] Install and verify Python.
- [ ] Create frontend Vite app.
- [ ] Create backend virtual environment.
- [ ] Install and pin backend dependencies.
- [ ] Configure local MongoDB.
- [ ] Configure ChromaDB.
- [ ] Configure LLM provider using environment variables.
- [ ] Confirm `.env` is ignored by Git.

**Blockers:**
- None recorded.

## 3. Backend foundation

- [ ] Create FastAPI application.
- [ ] Add `/health` endpoint.
- [ ] Add configuration loader.
- [ ] Add MongoDB connection and startup/shutdown handling.
- [ ] Add `/api/v1` router.
- [ ] Add request validation with Pydantic.
- [ ] Add consistent error response format.
- [ ] Add structured logging with secret redaction.
- [ ] Add backend tests.

## 4. Frontend foundation

- [ ] Create React app.
- [ ] Configure routing.
- [ ] Create shared layout and navigation.
- [ ] Create landing page.
- [ ] Create assistant page.
- [ ] Create scheme explorer page.
- [ ] Create document explorer page.
- [ ] Create API client and environment configuration.
- [ ] Add loading, empty, and error states.
- [ ] Add responsive layout and accessibility checks.

## 5. Official source and document pipeline

- [ ] Define source verification policy.
- [ ] Create `source_registry` data model.
- [ ] Select first official source(s).
- [ ] Verify source reuse/access terms.
- [ ] Add document metadata model.
- [ ] Implement safe PDF upload validation.
- [ ] Implement PDF text extraction.
- [ ] Preserve page numbers and section metadata.
- [ ] Add OCR fallback for scanned PDFs.
- [ ] Add text cleaning and chunking.
- [ ] Store document/chunk metadata.
- [ ] Add content hash and duplicate detection.
- [ ] Add document processing status and error handling.

## 6. Retrieval and AI assistant

- [ ] Select and configure embedding model.
- [ ] Create ChromaDB collection.
- [ ] Store chunk embeddings with stable metadata IDs.
- [ ] Implement semantic retrieval.
- [ ] Add keyword/hybrid retrieval if needed.
- [ ] Filter retrieval to approved/current sources.
- [ ] Implement answer generation using retrieved evidence only.
- [ ] Return source title, URL, page/section, and checked date.
- [ ] Validate citations against retrieved chunks.
- [ ] Implement insufficient-evidence response.
- [ ] Test hallucination and irrelevant-retrieval cases.
- [ ] Evaluate Hindi and English answers.

## 7. Schemes and eligibility

- [ ] Define scheme schema.
- [ ] Add verified scheme records.
- [ ] Add scheme search and filters.
- [ ] Define structured eligibility rule schema.
- [ ] Implement deterministic rule evaluator.
- [ ] Return passed, failed, and unknown conditions.
- [ ] Show official source for every rule.
- [ ] Add eligibility disclaimer.
- [ ] Test boundary values and missing profile fields.

## 8. Authentication, privacy, and security

- [ ] Implement registration/login/logout if required by MVP.
- [ ] Hash passwords with Argon2id or bcrypt.
- [ ] Use secure HttpOnly cookies for browser sessions.
- [ ] Add CSRF protection appropriate to cookie-based authentication.
- [ ] Restrict CORS to known frontend origins.
- [ ] Add rate limits to login, chat, and upload endpoints.
- [ ] Validate file type, size, and content.
- [ ] Keep uploads outside public/static directories.
- [ ] Enforce per-user resource access checks.
- [ ] Avoid collecting unnecessary personal information.
- [ ] Add retention/deletion policy for user data and uploads.
- [ ] Check logs for secrets and personal data.
- [ ] Run dependency and security checks.
- [ ] Review production HTTPS and cookie settings.

## 9. Language and accessibility

- [ ] Add Hindi interface labels.
- [ ] Add English interface labels.
- [ ] Add response language selection.
- [ ] Test Hindi retrieval and answer quality.
- [ ] Verify names, dates, amounts, and legal conditions survive translation.
- [ ] Add keyboard navigation and readable contrast.
- [ ] Test mobile and desktop layouts.

## 10. Testing and deployment

- [ ] Unit tests for services and rule evaluator.
- [ ] API integration tests.
- [ ] Frontend-to-backend end-to-end test.
- [ ] Access-control tests.
- [ ] File upload security tests.
- [ ] Citation correctness tests.
- [ ] Test unavailable LLM/database scenarios.
- [ ] Configure production environment variables.
- [ ] Deploy backend.
- [ ] Deploy frontend.
- [ ] Configure production CORS and HTTPS.
- [ ] Run smoke tests against deployed app.
- [ ] Prepare live demo using real official information.

## 11. Demo and submission

- [ ] Prepare one realistic citizen question.
- [ ] Confirm the source is official and current.
- [ ] Demonstrate Hindi and English.
- [ ] Demonstrate citations and page references.
- [ ] Demonstrate eligibility with missing information.
- [ ] Demonstrate honest uncertainty for unsupported questions.
- [ ] Update README setup instructions.
- [ ] Add architecture diagram.
- [ ] Add screenshots only from the working application.
- [ ] Prepare pitch deck.
- [ ] Prepare demo video/live walkthrough.
- [ ] Verify public repository has no secrets or private data.

---

## Current sprint

### Sprint goal
Create the smallest working full-stack foundation: React frontend calls FastAPI `/health`, and FastAPI connects to MongoDB.

### Tasks
- [ ] Frontend scaffold
- [ ] Backend scaffold
- [ ] Health endpoint
- [ ] Frontend health check
- [ ] MongoDB connection
- [ ] Commit working baseline

### Sprint notes
Write decisions, blockers, and links here.

---

## Completed work log

| Date | Work completed | Evidence (commit / PR / test) | Owner |
|---|---|---|---|
| YYYY-MM-DD | Project documentation created | Add commit/PR | Team |

## Bugs and blockers

| ID | Issue | Priority | Owner | Status | Notes |
|---|---|---|---|---|---|
| B-001 | Add issue here | Medium | Unassigned | Open | |

## Decisions log

| Date | Decision | Reason | Owner |
|---|---|---|---|
| YYYY-MM-DD | Use a modular FastAPI backend for MVP | Simpler to build, test, and deploy than microservices | Team |

## Weekly review

At the end of each work session, answer:

1. What actually works now?
2. What did we test?
3. What failed?
4. What is the next smallest task?
5. Did we introduce any security or privacy risk?

Do not mark tasks complete without evidence.
