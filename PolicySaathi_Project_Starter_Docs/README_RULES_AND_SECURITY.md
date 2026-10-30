# PolicySaathi — Development Rules, Scope Control & Security Standards

This document is the team's working agreement. Read it before creating features or changing architecture.

The objective is to build a reliable, secure, demonstrable product—not a large collection of unfinished screens.

---

## 1. Scope control: avoid unnecessary work

### MVP priority

Build in this order:

1. Real official document ingestion.
2. PDF text extraction with page references.
3. Retrieval from those documents.
4. AI answers grounded in retrieved passages.
5. Verifiable citations.
6. Hindi and English responses.
7. Scheme search using verified data.
8. Deterministic eligibility rules.
9. Security, tests, and deployment.

A feature is not part of the MVP merely because it sounds impressive.

### Do not build these unless the MVP works first

- Native Android/iOS apps.
- Microservices or Kubernetes.
- Complex admin analytics dashboards.
- Social feeds, likes, comments, or public profiles.
- Payment systems or subscriptions.
- Blockchain or token systems.
- Autonomous agents that submit government applications.
- Automatic scraping of every government website.
- Voice features before text answers and citations are reliable.
- Multiple LLM providers before one provider is working.
- A custom model trained from scratch.
- Elaborate animations that delay core workflows.
- A large notification/reminder system.
- A custom vector database.
- Multiple databases for the same type of data without a clear need.

### The feature test

Before starting a feature, answer:

1. Which citizen problem does it solve?
2. Is it required for the MVP or evaluation?
3. What real data/API/backend behaviour will make it work?
4. How will we test it?
5. What is the smallest useful version?

If these questions have no clear answers, add the idea to a “Later” list instead of implementing it now.

---

## 2. Engineering rules

- Prefer readable, small modules over giant files.
- Keep route handlers thin; put business logic in services.
- Validate all incoming data at the API boundary.
- Use consistent naming and response formats.
- Keep frontend API calls in a shared API layer.
- Do not duplicate business rules in React and FastAPI.
- Keep eligibility evaluation on the backend.
- Use database indexes for fields used in frequent lookups.
- Add pagination to potentially large list endpoints.
- Use explicit time zones and store timestamps consistently in UTC.
- Use migrations or documented scripts for schema/data changes.
- Pin dependency versions for reproducible builds.
- Do not add a package without a reason.
- Do not commit generated build folders, virtual environments, local databases, or uploads.
- Avoid premature abstractions; refactor when repetition or complexity is real.
- Do not create empty placeholder files for every planned feature.

### API rules

- Prefix application endpoints with `/api/v1`.
- Use correct HTTP methods and status codes.
- Validate path parameters, query parameters, and request bodies.
- Return safe, predictable error messages.
- Never return stack traces, secrets, password hashes, or internal configuration to clients.
- Add request IDs to logs to help debug failures.
- Apply pagination and maximum limits to search and list endpoints.
- Set timeouts for external API/model calls.
- Handle provider failures and rate limits gracefully.

### Git rules

- Use feature branches, for example `feature/document-upload`.
- Make small commits with meaningful messages.
- Never commit `.env`, API keys, credentials, private documents, or user data.
- Review diffs before pushing.
- Use pull requests for changes to authentication, permissions, file processing, and AI retrieval.
- Keep `PROGRESS.md` updated.
- Do not commit code that has not at least been run locally.

---

## 3. Security requirements

Security is part of each feature, not a final polishing task.

### 3.1 Secrets and configuration

- Store secrets only in environment variables or a managed secret store.
- Commit `.env.example` with empty placeholders, never real values.
- Add `.env`, private keys, certificates, local database files, and uploads to `.gitignore`.
- Use separate secrets for development and production.
- Generate strong random production secrets.
- Rotate any secret that is accidentally exposed.
- Do not put backend API keys in React/Vite environment variables. Anything bundled into frontend code is public.
- Do not print tokens, passwords, API keys, or full sensitive request bodies in logs.

### 3.2 Authentication

If accounts are included:

- Hash passwords with Argon2id (preferred) or bcrypt using a suitable work factor.
- Never store plaintext or reversibly encrypted passwords.
- Use generic login failure messages to reduce account enumeration.
- Add rate limiting and progressive delay to login and password-reset routes.
- Use short-lived access sessions and a secure refresh/session strategy.
- For browser authentication, prefer `HttpOnly`, `Secure` (in production), and appropriate `SameSite` cookies.
- Never store session tokens in `localStorage`.
- Invalidate sessions on logout and when credentials are reset.
- Consider session rotation and reuse detection for refresh tokens.
- Require re-authentication for sensitive account changes.

### 3.3 Authorisation and ownership

Authentication only proves who the user is. Authorisation decides what that user may access.

- Check ownership on every private resource request.
- Never trust a `userId` sent by the browser as proof of ownership.
- Derive the current user from the verified session.
- Query private records using both resource ID and authenticated user ID where applicable.
- Enforce admin permissions on the backend, not only by hiding frontend buttons.
- Deny access by default.
- Test that User A cannot read, modify, or delete User B's conversations, profile, uploads, or saved items.
- Do not expose private citizen profile fields in public search results.

### 3.4 Cookies, CORS, and CSRF

- Use HTTPS in production.
- Set session cookies as `HttpOnly` and `Secure` in production.
- Choose `SameSite=Lax` or `Strict` where compatible with the deployment design.
- If cross-site cookies are genuinely required, use `SameSite=None; Secure` and implement CSRF protection.
- Configure CORS with exact trusted frontend origins.
- Do not use wildcard origins with credentialed requests.
- Allow only required methods and headers.
- For cookie-authenticated state-changing requests, use CSRF tokens or another robust CSRF defence.
- Validate `Origin`/`Referer` as an additional defence where appropriate; do not rely on it alone.

### 3.5 File upload security

Government PDFs and user-uploaded files are untrusted input.

- Set a strict maximum file size.
- Allow only required file types.
- Do not trust filename extensions or browser-provided MIME types.
- Check file signatures/content and parse the file safely.
- Generate server-side random filenames; never use a user filename as a storage path.
- Store uploads outside the public web root.
- Prevent path traversal and archive/decompression bombs.
- Limit page count, processing time, memory, and OCR resources.
- Use malware scanning where available and appropriate.
- Reject encrypted, malformed, or unsupported documents with a safe error.
- Do not execute macros, scripts, embedded files, or links from uploaded documents.
- Do not render uploaded HTML or SVG as trusted application content.
- Delete temporary files after processing.
- Define retention and deletion rules for original uploads.
- Consider a separate low-privilege process/container for parsing untrusted documents.

### 3.6 AI and RAG security

Retrieved documents and user prompts are untrusted data.

- Treat document content as data, never as system instructions.
- Defend against prompt injection inside PDFs, web pages, and retrieved passages.
- Do not allow retrieved text to change system rules or grant tool permissions.
- Give the model only the tools and data required for the current task.
- Do not let the model execute arbitrary code, shell commands, database queries, or unrestricted URLs.
- Validate all tool arguments on the backend.
- Use allowlisted tools with explicit input/output schemas.
- Require user confirmation before any consequential external action.
- Do not allow the model to submit applications or change official records in the MVP.
- Separate retrieved evidence from the model's generated explanation.
- Generate citations from retrieval metadata, not from model-invented URLs.
- Verify that every citation points to a retrieved source and relevant page/section.
- If evidence is weak, conflicting, stale, or absent, say so.
- Never let the LLM invent eligibility rules, deadlines, benefit amounts, or application links.
- Keep deterministic eligibility logic separate from LLM prose generation.
- Record which document versions supported an answer so it can be audited.

### 3.7 Data privacy and minimisation

- Ask only for information needed for a specific feature.
- Make profile fields optional unless strictly necessary.
- Do not ask for Aadhaar, bank details, passwords, OTPs, or full identity documents for basic scheme discovery.
- Avoid collecting caste, religion, disability, health, or other sensitive information unless a clearly justified scheme rule requires it and the user explicitly chooses to provide it.
- Explain why a field is requested and how it will be used.
- Do not send unnecessary personal data to an LLM provider.
- Mask or remove personal data before model calls where possible.
- Do not use citizen questions or profiles for training without a clear lawful basis and informed consent.
- Provide a way to delete accounts and associated personal data, subject to applicable retention obligations.
- Define retention periods for uploads, conversations, logs, and backups.
- Restrict staff/admin access and log sensitive administrative actions.

### 3.8 API and infrastructure security

- Use HTTPS in production.
- Keep dependencies updated and scan for known vulnerabilities.
- Apply rate limits to login, upload, search, and AI endpoints.
- Set request body and upload limits.
- Set timeouts for database, HTTP, and model-provider calls.
- Validate external URLs and restrict server-side fetches to prevent SSRF.
- Do not fetch arbitrary user-provided URLs from the backend.
- Use least-privilege database credentials.
- Restrict database network access.
- Keep debug mode disabled in production.
- Add security headers such as Content-Security-Policy, `X-Content-Type-Options`, and an appropriate `Referrer-Policy`.
- Configure trusted proxy settings carefully.
- Back up production data and test restoration.
- Keep development, staging, and production environments separate.

### 3.9 Logging and monitoring

- Log security events, errors, request IDs, and service health.
- Do not log passwords, tokens, API keys, full uploaded documents, or unnecessary personal data.
- Restrict access to logs.
- Add alerts for repeated failed logins, unusual upload volume, and provider failures.
- Avoid storing full prompts/responses when they contain sensitive information; if stored for debugging, minimise, redact, restrict, and set retention.
- Keep an audit trail for source verification and admin changes.

---

## 4. Trust and accuracy rules for government information

- Prefer official government sources and primary documents.
- Record issuing department, official URL, publication date, effective date, retrieval date, jurisdiction, and version.
- Do not treat a news article, blog, social post, or AI answer as the legal source.
- Verify whether a document is a draft, notification, circular, amendment, superseded version, or currently effective rule.
- Do not assume a central scheme applies identically in every state.
- Show state/jurisdiction and date context.
- Display “last checked” information.
- Preserve source wording for critical legal conditions.
- Separate a plain-language explanation from the original legal text.
- Do not state that a user is officially eligible or approved.
- Direct users to the relevant official department/portal for final confirmation.
- If two official sources conflict, show the conflict and avoid choosing one without a defensible basis.

### Required answer behaviour

The assistant must:

1. Answer only to the extent supported by retrieved evidence.
2. Cite the official source and relevant page/section.
3. Distinguish facts from interpretation.
4. Identify missing facts that affect the answer.
5. Mention the effective date or last-verified date when relevant.
6. State uncertainty clearly.
7. Avoid legal guarantees or promises of benefits.

---

## 5. AI output and evaluation rules

Create a small evaluation set before expanding features.

For each test question, record:

- Question and language
- Expected official source
- Expected relevant passage/page
- Required facts
- Facts the assistant must not claim
- Whether the correct response should be “insufficient evidence”

Evaluate:

- Retrieval relevance
- Citation correctness
- Factual accuracy
- Unsupported-claim rate
- Hindi/English meaning preservation
- Handling of conflicting/outdated sources
- Correct eligibility result for known test profiles

Do not evaluate only by whether the answer sounds fluent. A fluent but unsupported answer is a failure.

---

## 6. Code review checklist

Before merging a pull request:

- [ ] Does this solve an agreed user problem?
- [ ] Is this the smallest useful implementation?
- [ ] Are inputs validated?
- [ ] Are permissions checked on the backend?
- [ ] Are secrets and personal data protected?
- [ ] Are external calls bounded by timeouts and error handling?
- [ ] Are database queries scoped to the authenticated user?
- [ ] Are citations generated from actual retrieved evidence?
- [ ] Are tests included?
- [ ] Does the feature have loading, empty, and error states?
- [ ] Is `PROGRESS.md` updated?
- [ ] Are docs and `.env.example` updated if configuration changed?

---

## 7. Release checklist

Do not deploy a public demo until:

- [ ] No secrets are present in Git history or frontend bundles.
- [ ] Production CORS contains only approved origins.
- [ ] HTTPS is enabled.
- [ ] Production cookies have secure settings.
- [ ] Debug mode is disabled.
- [ ] Database credentials are least-privilege.
- [ ] Authentication and ownership tests pass.
- [ ] Upload validation and size limits are enabled.
- [ ] Rate limits are enabled.
- [ ] Logs redact sensitive values.
- [ ] Privacy notice and AI limitations are visible.
- [ ] The assistant handles unsupported questions safely.
- [ ] Official sources and citations have been checked.
- [ ] A backup/restore plan exists for persistent data.

---

## 8. Non-negotiable team rules

1. **No fake data presented as real.** Demo data must be clearly labelled as sample.
2. **No fake citations.** A source link must support the claim.
3. **No hardcoded API keys or passwords.**
4. **No frontend-only security.** Enforce permissions in the backend.
5. **No unnecessary personal data collection.**
6. **No autonomous application submission in the MVP.**
7. **No feature marked complete without testing.**
8. **No scope expansion while core MVP tasks are broken.**
9. **No unreviewed changes to authentication, uploads, or authorisation.**
10. **When uncertain, disclose uncertainty rather than guessing.**

If a shortcut conflicts with these rules, choose the safer implementation and reduce feature scope instead.
