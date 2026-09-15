# Easy Resume integration contract

Easy Resume is local-first. The browser stores the career workspace, while optional server endpoints provide AI generation and Credly retrieval without exposing provider credentials to the client.

## Source-of-truth invariant

`master` is the canonical career profile.

Generated or imported data must **never** silently overwrite master data. Derived artifacts record `masterRevision` / `sourceRevision`. Credential imports are staged as candidates and require explicit user acceptance followed by a master save.

## POST `/api/easy-resume/ai`

### Request

```json
{
  "task": "tailor | cover | interview",
  "masterRevision": 7,
  "payload": {
    "role": "Azure Solutions Architect",
    "company": "Example Ltd",
    "jobDescription": "...",
    "tone": "Professional"
  }
}
```

The server should receive the minimum necessary profile context from an authenticated or explicitly submitted request. Do not persist resume content unless the product has a documented retention policy and the user has consented.

### Tailor response

```json
{
  "summary": "Evidence-constrained targeted summary",
  "keywords": ["azure", "architecture"],
  "missing": ["terraform"],
  "note": "No unsupported claims introduced"
}
```

Rules:

- Only reuse claims supported by the master profile supplied for the request.
- Never invent employers, dates, qualifications, credentials, metrics, revenue, headcount, project outcomes or technologies.
- Missing keywords must remain gaps unless evidence exists in the master profile.
- Keep generated content ATS-readable and plain-text compatible.

### Cover response

```json
{
  "text": "Dear Hiring Team, ..."
}
```

Rules:

- Base claims on master-profile evidence.
- Job/company context may influence emphasis but not create new facts.
- Do not claim familiarity with a company, product or project unless supplied in the request.

### Interview response

```json
{
  "questions": ["Tell us about..."],
  "matched": ["azure"],
  "missing": ["terraform"]
}
```

Questions may probe missing skills, but suggested answers should never be fabricated. Encourage STAR answers using real examples.

## GET `/api/easy-resume/credly?url=<encoded-profile-url>`

This endpoint acts as a same-origin server proxy for an explicitly supplied public Credly profile URL.

Expected response:

```json
{
  "credentials": [
    {
      "name": "Credential name",
      "issuer": "Issuer",
      "credentialId": "optional-id",
      "issueDate": "2026-01-01",
      "expiryDate": "2029-01-01",
      "url": "https://www.credly.com/...",
      "source": "Credly"
    }
  ]
}
```

Security and validation:

- Allow only HTTPS Credly domains approved by the application.
- Protect against SSRF; do not accept arbitrary proxy destinations.
- Validate response content types and enforce response-size/time limits.
- Do not treat imported data as trusted master data until the user explicitly reviews it.
- Preserve the verification URL and issuer as provenance.

## Client fallback behaviour

If either endpoint is unavailable:

- AI features use the local evidence-constrained assistant.
- Credly web import instructs the user to import a local JSON/CSV credential export.
- Core resume building, ATS matching, application tracking, backup and export remain functional.

## Recommended production controls

- Authentication for server AI calls.
- Per-user and per-IP rate limiting.
- CSRF protections where applicable.
- Structured request validation.
- Audit metadata without logging full resume/job-description bodies by default.
- Explicit model/provider configuration server-side.
- Content-retention policy and user-facing privacy disclosure.
- Encryption in transit and at rest for any persisted career data.
