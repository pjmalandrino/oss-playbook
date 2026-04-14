# Security Checklist

> Standalone reference checklist. Used during [code review](../processes/code-review.md), [quality audits](../processes/audit.md), and [security response](../processes/security-response.md).

## Input validation

- [ ] All API inputs validated via Pydantic schemas (type, length, format)
- [ ] File uploads validated: MIME type whitelist, max size (`MAX_FILE_SIZE_MB`), page count limit
- [ ] Path traversal prevented: no user input in file paths without sanitization
- [ ] Query parameters validated: no raw SQL or NoSQL injection vectors
- [ ] Request body size limited at framework/nginx level

## Output encoding

- [ ] All dynamic HTML sanitized with DOMPurify (frontend)
- [ ] Markdown rendered with Marked + DOMPurify (never `v-html` on raw user content)
- [ ] API responses use Pydantic serialization (no raw dict dumps)
- [ ] Error messages don't leak internal details (stack traces, file paths, SQL)

## Authentication & authorization

- [ ] API keys not hardcoded (use environment variables)
- [ ] CORS origins explicitly configured (`CORS_ORIGINS` env var)
- [ ] Rate limiting enforced (`RATE_LIMIT_RPM` per IP)
- [ ] No sensitive data in URL query parameters (use headers/body)

## Secrets management

- [ ] No secrets in source code (grep for `password`, `token`, `secret`, `key`)
- [ ] `.env` file in `.gitignore`
- [ ] `.env.example` contains placeholder values only
- [ ] Docker images don't contain `.env` or credentials
- [ ] Git history clean of accidentally committed secrets

## Dependencies

- [ ] `pip-audit` clean (no known vulnerabilities in Python deps)
- [ ] `npm audit` clean (no critical/high vulnerabilities in Node deps)
- [ ] Trivy image scan clean (no critical vulnerabilities in Docker image)
- [ ] Dependencies pinned to version ranges (not `*` or `latest`)

## Infrastructure

- [ ] Nginx configured with security headers (X-Frame-Options, CSP, etc.)
- [ ] Docker runs as non-root user (`appuser`)
- [ ] OpenSearch security disabled only in dev (never in production)
- [ ] Health endpoints don't expose sensitive system information

## OWASP Top 10 quick check

| # | Risk | Our mitigation |
|---|------|----------------|
| A01 | Broken Access Control | CORS + rate limiting + input validation |
| A02 | Cryptographic Failures | No sensitive data stored unencrypted |
| A03 | Injection | Pydantic validation + parameterized queries (aiosqlite) |
| A04 | Insecure Design | Hexagonal architecture isolates trust boundaries |
| A05 | Security Misconfiguration | `.env.example` + documented defaults |
| A06 | Vulnerable Components | `pip-audit` + `npm audit` + Trivy in release gate |
| A07 | Auth Failures | Rate limiting + no default credentials |
| A08 | Data Integrity Failures | Signed Docker images + CI verification |
| A09 | Logging Failures | Structured logging, no sensitive data in logs |
| A10 | SSRF | External URL validation in converter adapters |
