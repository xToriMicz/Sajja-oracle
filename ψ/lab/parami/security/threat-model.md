# Threat Model — Static Landing Page + API

> รู้จักศัตรู รู้จักตัวเอง — ร้อยศึก ไม่เคยแพ้

## System Overview

```
┌─────────────┐     HTTPS     ┌─────────────┐     Internal     ┌──────────┐
│   Browser   │ ────────────► │  Static CDN │                  │          │
│   (User)    │               │  / Hosting  │                  │    DB    │
└─────────────┘               └─────────────┘                  │          │
       │                                                       └──────────┘
       │          HTTPS/API                                         ▲
       └──────────────────────► ┌─────────────┐                    │
                                │   API Server│ ───────────────────┘
                                └─────────────┘
```

## Assets

| Asset | Sensitivity | Location |
|---|---|---|
| Static HTML/CSS/JS | Low | CDN / Hosting |
| API credentials / keys | Critical | Server env / secrets |
| User data (if any) | High | Database |
| Session tokens | High | Client cookies / headers |
| Admin access | Critical | API / Hosting dashboard |

## Threat Actors

| Actor | Motivation | Capability |
|---|---|---|
| Script kiddie | Defacement, bragging | Automated scanners, known exploits |
| Competitor | Data theft, disruption | Moderate technical skill |
| Insider | Sabotage, data exfiltration | Full system access |
| Automated bot | Credential stuffing, spam | High volume, low sophistication |

## Threats — Static Landing Page

### T1: XSS via DOM Manipulation
- **Vector**: Malicious URL parameters reflected in page without sanitization
- **Impact**: Cookie theft, phishing overlay, redirect to malicious site
- **Likelihood**: Medium (common if JS reads URL params)
- **Mitigation**: CSP header, no `innerHTML` with untrusted data, input encoding

### T2: Clickjacking
- **Vector**: Page embedded in attacker's iframe with invisible overlay
- **Impact**: User tricked into clicking unintended actions
- **Likelihood**: Low (static page has limited actions)
- **Mitigation**: `X-Frame-Options: DENY`, `frame-ancestors 'none'` in CSP

### T3: Content Injection / Defacement
- **Vector**: Compromised CDN or hosting account
- **Impact**: Brand damage, malware distribution to visitors
- **Likelihood**: Low
- **Mitigation**: 2FA on hosting, SRI (Subresource Integrity) for external scripts, CSP

### T4: Mixed Content / Downgrade Attack
- **Vector**: HTTP resources loaded on HTTPS page
- **Impact**: Man-in-the-middle injection of malicious content
- **Likelihood**: Low (if HSTS is configured)
- **Mitigation**: HSTS header, all resources loaded over HTTPS

### T5: Third-Party Script Compromise
- **Vector**: Compromised CDN-hosted library (e.g., analytics, fonts)
- **Impact**: Supply chain attack — arbitrary code execution in user's browser
- **Likelihood**: Medium
- **Mitigation**: SRI hashes on `<script>` and `<link>` tags, CSP with specific sources

## Threats — API

### T6: Injection (SQL / NoSQL / Command)
- **Vector**: Unsanitized input passed to database queries or system commands
- **Impact**: Data breach, data destruction, remote code execution
- **Likelihood**: High (if not using parameterized queries)
- **Mitigation**: Parameterized queries, input validation, least-privilege DB user

### T7: Broken Authentication
- **Vector**: Weak passwords, missing rate limiting, token leakage
- **Impact**: Unauthorized access to API and data
- **Likelihood**: Medium
- **Mitigation**: Strong password policy, rate limiting, secure token storage, short expiry

### T8: CSRF on API Endpoints
- **Vector**: Forged requests from attacker's site using authenticated session
- **Impact**: Unauthorized state changes (data modification, deletion)
- **Likelihood**: Medium (if using cookies for auth)
- **Mitigation**: CSRF tokens, `SameSite` cookies, custom request headers

### T9: CORS Misconfiguration
- **Vector**: Overly permissive `Access-Control-Allow-Origin`
- **Impact**: Unauthorized cross-origin data access
- **Likelihood**: Medium
- **Mitigation**: Whitelist specific origins, never use `*` with credentials

### T10: Rate Limiting / DoS
- **Vector**: Flood of requests to API endpoints
- **Impact**: Service unavailability, increased infrastructure cost
- **Likelihood**: Medium
- **Mitigation**: Rate limiting per IP/user, request size limits, CDN-level protection

### T11: Sensitive Data Exposure
- **Vector**: API returns excessive data, errors expose stack traces
- **Impact**: Information leakage aiding further attacks
- **Likelihood**: Medium
- **Mitigation**: Minimal response payloads, generic error messages in production, no debug info

### T12: Dependency Vulnerabilities
- **Vector**: Known CVEs in npm/pip/etc packages
- **Impact**: Varies — from DoS to RCE depending on the vulnerability
- **Likelihood**: High (dependencies go stale)
- **Mitigation**: `npm audit` / `pip-audit` in CI, Dependabot/Renovate, lock files

## Risk Matrix

| Threat | Likelihood | Impact | Priority |
|---|---|---|---|
| T1: XSS | Medium | High | **High** |
| T5: 3rd-party scripts | Medium | High | **High** |
| T6: Injection | High | Critical | **Critical** |
| T7: Broken Auth | Medium | High | **High** |
| T12: Dependencies | High | Medium | **High** |
| T9: CORS misconfig | Medium | Medium | **Medium** |
| T10: Rate limiting | Medium | Medium | **Medium** |
| T11: Data exposure | Medium | Medium | **Medium** |
| T8: CSRF | Medium | Medium | **Medium** |
| T2: Clickjacking | Low | Low | **Low** |
| T3: Defacement | Low | Medium | **Low** |
| T4: Mixed content | Low | Low | **Low** |

## Next Steps

1. Run audit checklist against current codebase (see `audit-checklist.md`)
2. Implement recommended headers (see `recommended-headers.md`)
3. Set up automated dependency scanning in CI
4. Schedule periodic security review
