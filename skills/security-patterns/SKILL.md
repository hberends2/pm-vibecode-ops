---
name: security-patterns
description: MUST activate when writing: authentication/authorization code, login/session management, API endpoints handling user data, database queries, input validation, file uploads, external API calls, password handling, token generation, error responses. Enforces OWASP Top 10 prevention: broken access control, cryptographic failures, injection, insecure design, security misconfiguration, vulnerable components, authentication failures, data integrity failures, logging failures, SSRF.
---

# Security Patterns

Shift security left - prevent vulnerabilities while writing code, not at review time.

## Enforcement Checklist

When writing security-relevant code:

1. **AUTH**: Every protected endpoint has authentication
2. **AUTHZ**: Every data access has authorization check
3. **INPUT**: All external input is validated
4. **QUERIES**: All database queries are parameterized
5. **SECRETS**: All secrets from env, never hardcoded
6. **ERRORS**: No sensitive data in responses
7. **LOGGING**: Security events logged, PII excluded

## Quick Reference

| Vulnerability | Prevention |
|--------------|------------|
| Broken Access Control | Auth + authz on every endpoint |
| Cryptographic Failures | argon2id/bcrypt, randomBytes for tokens |
| Injection | Parameterized queries only |
| Insecure Design | Rate limiting, account lockout |
| Security Misconfiguration | Helmet headers, secure cookies |
| Vulnerable Components | npm audit, fail CI on critical |
| Authentication Failures | timingSafeEqual for comparisons |
| Data Integrity | Verify webhook signatures |
| Logging Failures | Log events, never log PII/secrets |
| SSRF | URL allowlist, block internal IPs |

## Core Principles

- **Fail secure**: Deny by default, allow explicitly
- **Defense in depth**: Multiple layers of protection
- **Least privilege**: Minimal permissions required
- **Secure defaults**: Safe configuration out of the box

See `references/owasp-patterns.md` for detailed code examples for each OWASP Top 10 vulnerability.

**This skill applies to: passwords, tokens, user data, external APIs, database queries, file uploads.**
