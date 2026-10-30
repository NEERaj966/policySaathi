# PolicySaathi Database Notes

## Core collections

### users
Account identity and authentication fields. Never store plaintext passwords.

### citizen_profiles
Optional user-provided attributes used for scheme filtering or eligibility. Keep sensitive fields out unless a specific feature genuinely requires them.

### documents
Official document metadata: title, issuer, canonical URL, jurisdiction, language, dates, version/status, verification status, content hash, and processing state.

### document_chunks
Optional MongoDB metadata records for extracted chunks. Each chunk references `documentId` and stores page/section, text or a safe text reference, and the corresponding vector ID.

### schemes
Structured scheme details and official links.

### eligibility_rules
Structured conditions linked to `schemeId`, with source document and page/section references.

### conversations and messages
A conversation belongs to one user. A message belongs to one conversation. Scope every query by the authenticated owner.

### saved_schemes
Join collection for the many-to-many relationship between users and schemes. Add a unique compound index on `(userId, schemeId)`.

### source_registry
Approved source organisations/domains and verification metadata.

### document_versions
Version history for documents and policy changes.

## Relationship summary

- users 1 — 0..1 citizen_profiles
- users 1 — many conversations
- conversations 1 — many messages
- documents 1 — many document_chunks
- schemes 1 — many eligibility_rules
- users many — many schemes through saved_schemes
- schemes many — many documents when a join collection such as scheme_sources is required

## Index suggestions

Create indexes based on real query patterns, for example:
- `users.email` unique
- `citizen_profiles.userId` unique
- `conversations.userId, updatedAt`
- `messages.conversationId, createdAt`
- `documents.status, jurisdiction, publishedAt`
- `schemes.state, category, status`
- `eligibility_rules.schemeId`
- `saved_schemes.userId, schemeId` unique

Review indexes against actual usage; do not add every possible index blindly.
