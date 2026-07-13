# Security Rules

## Purpose
- Define baseline security expectations for code, data handling, and dependencies across the system.

## Core Principles
- Validate and sanitize all input at trust boundaries; never trust client-supplied data.
- Apply least privilege everywhere: users, service accounts, tokens, and database roles.
- Fail closed: default to denying access when a check is ambiguous or fails.
- Keep security-relevant logic (auth, authorization, crypto) centralized, not duplicated per endpoint.

## Secrets and Credentials
- Never commit secrets, API keys, tokens, or credentials to the repository.
- Load secrets from environment variables or a secrets manager, never hardcode them.
- Do not log secrets, tokens, passwords, or full credential objects.
- Rotate any secret that is accidentally committed or exposed; treat the old value as compromised.
- Use `.env` files only for local development and keep them out of version control.

## Input Validation and Injection Prevention
- Use parameterized queries or an ORM's safe query builder; never concatenate raw input into SQL.
- Escape or encode output based on context (HTML, URL, JS, shell) to prevent XSS.
- Never pass unsanitized input to a shell, `eval`, or dynamic code execution.
- Validate file uploads: restrict type, size, and storage location; never trust client-provided MIME type or filename.
- Reject unexpected fields and enforce strict schema validation on API payloads.

## Authentication and Authorization
- Enforce authentication on every endpoint except explicitly public routes.
- Check authorization (org/resource/role ownership) on every request, not just authentication.
- Never rely on client-side checks or hidden UI elements as the sole access control.
- Use short-lived tokens with refresh flows; avoid long-lived static tokens where possible.
- Hash passwords with a strong, salted algorithm (for example, bcrypt, argon2); never store plaintext or reversible-encrypted passwords.
- Invalidate sessions/tokens on logout, password change, and privilege change.

## Data Protection
- Encrypt sensitive data in transit (TLS) and at rest where required by data sensitivity.
- Apply the principle of least data: only collect, store, and return fields that are actually needed.
- Redact or mask sensitive fields (PII, tokens, secrets) in logs, error messages, and API responses per [[error-handling-rules]].
- Enforce org/tenant isolation on every query; never allow cross-tenant data access through missing `WHERE` scoping.

## Dependency and Supply Chain Security
- Keep dependencies up to date; prioritize patching known vulnerabilities (CVEs).
- Review new dependencies for maintenance status, license, and necessity before adding them.
- Pin dependency versions in lockfiles; avoid unpinned ranges in production manifests.
- Do not install or run packages from untrusted or unverified sources.

## Logging and Monitoring
- Log authentication failures, authorization failures, and other security-relevant events.
- Include enough context (`requestId`, `org`, actor, operation) to investigate incidents, per [[error-handling-rules]].
- Never log request bodies or headers containing credentials, tokens, or secrets.
- Alert on repeated auth failures, privilege escalation attempts, or anomalous access patterns.

## Infrastructure and Configuration
- Disable verbose error output (stack traces, debug pages) in production.
- Set secure HTTP headers (CSP, `X-Content-Type-Options`, `X-Frame-Options`, HSTS) where applicable.
- Restrict CORS to known, explicit origins; avoid wildcard origins for authenticated endpoints.
- Keep production configuration and credentials separate from development/test environments.

## Testing Requirements
- Add tests for authorization boundaries (a user cannot access another org's/user's data).
- Add regression tests for any fixed security vulnerability.
- Include negative test cases: invalid tokens, expired sessions, malformed/malicious input.
- Run dependency vulnerability scans as part of CI where available.

## Incident Response
- If a vulnerability or exposure is discovered, prioritize containment (rotate secrets, patch, disable affected endpoint) before root-cause analysis.
- Document the cause and fix once resolved; do not leave temporary mitigations undocumented.
