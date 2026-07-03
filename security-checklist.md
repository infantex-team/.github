# Security Checklist (OWASP-aligned)

> **Organization canonical source:** [infantex-team/.github](https://github.com/infantex-team/.github/blob/main/security-checklist.md)
>
> - **All projects:** use the [Quick PR checklist](#quick-pr-checklist) on security-sensitive PRs (org default PR template links here).
> - **Non-AIDE projects:** optionally copy to `docs/security-checklist.md` in each repo.
> - **AIDE hubs:** sync into `knowledge-base/conventions/security-checklist.md` and run `kb-index.sh`.
>
> Aligned with [OWASP Top 10 (2021)](https://owasp.org/Top10/) and [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/).
> Document project-specific auth and data classification in your architecture doc (AIDE: `knowledge-base/architecture/README.md` → Security Architecture).

---

## When to use

| Phase | Who | Action |
|-------|-----|--------|
| **Spec / design** | BA, architect, tech lead | Complete relevant rows before approval; fill spec **§6 Security Considerations** |
| **Implementation** | Developer, AI draft | Apply controls while coding; never skip auth/validation on new surfaces |
| **PR review** | Reviewer | Complete **Quick PR checklist** below |
| **Release** | Tech lead, DevOps | Verify config, secrets, dependencies, monitoring for affected components |

---

## Quick PR checklist

Use for every PR that touches auth, APIs, data handling, infra, or dependencies.

- [ ] **Access control:** Every new/changed endpoint or action checks authorization (not only authentication).
- [ ] **Input validation:** All external input validated at trust boundaries (type, length, format, allow-list).
- [ ] **Injection:** Queries use parameterization/ORM; no string-concatenated SQL/shell/commands.
- [ ] **Output encoding:** User-controlled data encoded appropriately in HTML/JSON/logs (XSS prevention).
- [ ] **Secrets:** No credentials, tokens, or private keys in code, comments, tests, or commits.
- [ ] **Crypto:** Sensitive data encrypted in transit (TLS) and at rest where required; no custom crypto.
- [ ] **Errors:** Error responses do not leak stack traces, internal paths, or secrets to clients.
- [ ] **Dependencies:** New/updated packages reviewed; known critical CVEs addressed or accepted with record.
- [ ] **Logging:** Security-relevant events logged without logging secrets or excessive PII.
- [ ] **Tests:** At least one negative/security test case for new auth, validation, or data-access logic.

---

## OWASP Top 10 — detailed checklist

### A01 — Broken Access Control

- [ ] Deny by default; grant least privilege (roles, scopes, row-level rules).
- [ ] Server-side enforcement on every request — never rely on UI hiding alone.
- [ ] Object-level authorization: users cannot access others' resources by ID tampering (BOLA/IDOR).
- [ ] CORS, file storage, and admin functions restricted to intended principals.
- [ ] Rate limiting on sensitive operations (login, password reset, exports) where applicable.

**Spec §6 must state:** roles allowed, forbidden cases, and how horizontal/vertical escalation is prevented.

### A02 — Cryptographic Failures

- [ ] TLS for all production traffic; no sensitive data over unencrypted channels.
- [ ] Passwords hashed with a modern algorithm (e.g. Argon2, bcrypt, scrypt) — never plaintext or reversible encoding.
- [ ] Encryption keys and certificates managed via secrets manager / env — not in source control.
- [ ] No deprecated algorithms (MD5, SHA1 for security, ECB mode, weak TLS versions).
- [ ] PII/PHI classification documented; retention and encryption match policy.

### A03 — Injection

- [ ] Parameterized queries or trusted ORM APIs for all database access.
- [ ] No OS/shell command construction from user input; use safe APIs or strict allow-lists.
- [ ] LDAP, XPath, template, and NoSQL queries sanitized or parameterized per platform guidance.
- [ ] File uploads: validate type/size, store outside web root, scan if policy requires.

### A04 — Insecure Design

- [ ] Threats considered for the feature (STRIDE or lightweight abuse cases).
- [ ] Business logic abuse cases documented (e.g. coupon stacking, race conditions, limit bypass).
- [ ] Separation of duties for high-risk actions (approval workflows, dual control) where required.
- [ ] Security requirements are testable acceptance criteria, not vague statements.

### A05 — Security Misconfiguration

- [ ] Default credentials removed; debug endpoints disabled in production.
- [ ] Security headers configured (CSP, HSTS, X-Content-Type-Options, etc.) per app type.
- [ ] Minimal attack surface: unused features, ports, and cloud permissions disabled.
- [ ] IaC and container images follow hardened baselines; secrets not baked into images.
- [ ] Environment-specific config via env/secrets — not committed `.env` with real values.

### A06 — Vulnerable and Outdated Components

- [ ] Dependency versions pinned or lockfiles committed.
- [ ] Automated scanning (Dependabot, Snyk, `npm audit`, etc.) enabled or run in CI.
- [ ] Critical/high CVEs triaged before release; exceptions documented with expiry.
- [ ] Only necessary dependencies added; supply-chain risk considered for new packages.

### A07 — Identification and Authentication Failures

- [ ] MFA available or required per project policy for privileged accounts.
- [ ] Session IDs unpredictable, HttpOnly, Secure, SameSite as appropriate; rotation on login.
- [ ] Account enumeration prevented in login/password-reset flows where feasible.
- [ ] Brute-force and credential-stuffing mitigations (rate limit, lockout, CAPTCHA).
- [ ] JWT/OAuth: validate issuer, audience, expiry, signature; short-lived tokens; secure storage on clients.

**Project-specific auth model:** document in your architecture security section.

### A08 — Software and Data Integrity Failures

- [ ] CI/CD pipelines protected; only trusted actors can deploy or change workflows.
- [ ] Artifacts signed or checksum-verified where supported.
- [ ] Deserialization of untrusted data avoided or strictly constrained.
- [ ] Webhooks and callbacks verified (signatures, shared secrets, replay protection).

### A09 — Security Logging and Monitoring Failures

- [ ] Authentication success/failure, authorization failures, and admin actions logged.
- [ ] Logs do not contain passwords, tokens, full payment data, or unnecessary PII.
- [ ] Correlation IDs for tracing incidents across services.
- [ ] Alerts or review process for anomalous patterns (defined per environment).

### A10 — Server-Side Request Forgery (SSRF)

- [ ] Outbound requests triggered by user input use allow-listed hosts/schemes or a proxy.
- [ ] Metadata endpoints (169.254.169.254, internal IPs) not reachable from user-controlled URLs.
- [ ] Redirect followers and URL fetchers validate destination before request.

---

## Cross-cutting requirements

### Data privacy

- [ ] Collect minimum data required; purpose documented.
- [ ] Data subject rights and retention aligned with legal/policy requirements.
- [ ] Cross-border transfer constraints noted if applicable.

### API security

- [ ] Authentication mechanism consistent with project standards.
- [ ] Pagination and filters cannot be abused for mass data exfiltration without authorization.
- [ ] Versioning and deprecation do not remove auth checks on legacy routes still in use.

### Frontend / mobile (if applicable)

- [ ] CSRF protection on state-changing browser requests (cookies/session).
- [ ] Sensitive data not stored in localStorage without encryption/risk acceptance.
- [ ] Deep links and WebViews do not bypass auth.

### Infrastructure

- [ ] Network segmentation and security groups follow least privilege.
- [ ] Backups encrypted; restore tested periodically.
- [ ] Incident response contact and severity rubric known to the team.

---

## Mapping to specification §6

When writing **Security Considerations** in a spec (AIDE or otherwise), address at minimum:

1. **Actors & assets** — who accesses what data/actions.
2. **AuthN/AuthZ** — mechanism and enforcement points (A01, A07).
3. **Input/output** — validation, encoding, injection risks (A03).
4. **Data protection** — classification, transit/rest, retention (A02).
5. **Abuse cases** — logic flaws and rate limits (A04).
6. **Dependencies & config** — new packages, env flags, defaults (A05, A06).
7. **Logging & monitoring** — what to log and alert on (A09).
8. **Out of scope** — explicit security exclusions with risk owner.

Mark **N/A** with one-line justification for categories that do not apply.

---

## References

- [OWASP Top 10 (2021)](https://owasp.org/Top10/)
- [OWASP Application Security Verification Standard (ASVS)](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/)
