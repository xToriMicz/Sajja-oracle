# Security Audit Checklist

> สัจจะ — ตรวจสอบความจริง ไม่ใช่ความเชื่อ

## XSS (Cross-Site Scripting)

- [ ] All user input is sanitized before rendering in HTML
- [ ] No use of `innerHTML`, `document.write()`, or `eval()` with untrusted data
- [ ] Template engines use auto-escaping (e.g., React JSX, Jinja2 `|e`)
- [ ] URL parameters are not reflected into page without encoding
- [ ] `Content-Type` headers are set correctly on all responses
- [ ] SVG/image uploads do not contain embedded scripts
- [ ] CSP header blocks inline scripts (`unsafe-inline` not used)

## CSRF (Cross-Site Request Forgery)

- [ ] All state-changing requests require CSRF tokens
- [ ] CSRF tokens are unique per session and unpredictable
- [ ] `SameSite` cookie attribute is set to `Strict` or `Lax`
- [ ] Custom headers required for API requests (e.g., `X-Requested-With`)
- [ ] GET requests do not perform state-changing operations

## Injection

- [ ] SQL queries use parameterized statements / prepared queries
- [ ] No string concatenation in database queries
- [ ] OS command execution avoids shell interpolation (`execFile` over `exec`)
- [ ] LDAP/NoSQL queries sanitize special characters
- [ ] File paths are validated against directory traversal (`../`)
- [ ] Log entries sanitize user input to prevent log injection

## CSP Headers (Content Security Policy)

- [ ] CSP header is present on all HTML responses
- [ ] `default-src 'self'` is set as baseline
- [ ] `script-src` does not include `'unsafe-inline'` or `'unsafe-eval'`
- [ ] `style-src` restricts to `'self'` and trusted CDNs with nonces
- [ ] `img-src` is restricted to known domains
- [ ] `frame-ancestors` prevents clickjacking (replaces X-Frame-Options)
- [ ] `report-uri` or `report-to` is configured for violation monitoring

## CORS Policy

- [ ] `Access-Control-Allow-Origin` is not set to `*` for authenticated endpoints
- [ ] Allowed origins are explicitly whitelisted
- [ ] `Access-Control-Allow-Credentials` is only used with specific origins
- [ ] Preflight responses have appropriate `Access-Control-Max-Age`
- [ ] `Access-Control-Allow-Methods` is restricted to required methods only
- [ ] `Access-Control-Allow-Headers` does not over-expose custom headers

## Input Sanitization

- [ ] All inputs are validated on the server side (client-side is supplementary)
- [ ] Input length limits are enforced
- [ ] Email, URL, and other structured inputs are validated against format
- [ ] File uploads are validated by content type, not just extension
- [ ] File upload size limits are enforced
- [ ] Rich text input uses allowlist-based sanitization (e.g., DOMPurify)
- [ ] JSON/XML parsers have depth and size limits to prevent DoS
- [ ] Unicode and encoding attacks are handled (null bytes, overlong UTF-8)
