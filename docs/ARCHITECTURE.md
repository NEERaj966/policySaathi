# PolicySaathi Architecture Notes

## Architectural approach

Use a modular monolith for the MVP. One FastAPI deployment contains separate API, service, repository, and integration modules.

## Main flows

### Document ingestion

Official source -> source validation -> document metadata -> PDF validation -> text extraction/OCR -> page-aware chunking -> embeddings -> ChromaDB. Store document metadata, processing state, and stable chunk identifiers in MongoDB.

### Question answering

User question -> authentication/rate limit -> language and intent detection -> retrieve relevant chunks -> filter by source status/jurisdiction/date -> generate answer from evidence -> validate citation IDs -> return answer and citations.

### Eligibility

Citizen-provided optional profile values -> retrieve verified scheme -> load structured rules -> deterministic evaluation -> return passed/failed/unknown conditions -> LLM may explain the result, but must not alter the calculation.

## Trust boundaries

- Browser input is untrusted.
- Uploaded documents are untrusted.
- Retrieved text is untrusted content, not instructions.
- LLM output is untrusted until citations and claims are checked.
- Only backend services may access database credentials and LLM secrets.

## Key design decisions

- MongoDB stores application records and document metadata.
- ChromaDB stores vectors; vector metadata includes stable document/chunk IDs.
- Keep page and section metadata through extraction and chunking.
- Do not split the application into microservices before there is a measured reason.
