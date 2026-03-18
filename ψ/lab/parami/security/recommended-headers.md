# Recommended Security Headers

> ป้องกันก่อน ดีกว่าแก้ทีหลัง

## Required Headers

### Content-Security-Policy (CSP)

```
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'nonce-{random}'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
```

**Why**: Primary defense against XSS. Controls which resources the browser is allowed to load.

### X-Frame-Options

```
X-Frame-Options: DENY
```

**Why**: Prevents clickjacking by blocking the page from being embedded in iframes. Use `SAMEORIGIN` if iframes are needed internally.

### X-Content-Type-Options

```
X-Content-Type-Options: nosniff
```

**Why**: Prevents MIME-type sniffing. Forces the browser to respect the declared `Content-Type`, blocking attacks where a file is served as a different type.

### Referrer-Policy

```
Referrer-Policy: strict-origin-when-cross-origin
```

**Why**: Controls how much referrer information is sent with requests. Prevents leaking internal URLs and query parameters to third parties.

### Strict-Transport-Security (HSTS)

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

**Why**: Forces HTTPS connections. Prevents protocol downgrade attacks and cookie hijacking over HTTP.

### Permissions-Policy

```
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
```

**Why**: Restricts browser features that the page can use. Reduces attack surface by disabling unnecessary APIs.

### X-XSS-Protection

```
X-XSS-Protection: 0
```

**Why**: Disables the legacy XSS auditor (which can introduce vulnerabilities). CSP is the modern replacement. Setting to `0` is recommended by OWASP when CSP is in place.

## Implementation Examples

### Static Site (Netlify / Vercel)

**Netlify** — `_headers` file:
```
/*
  Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; frame-ancestors 'none'
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
```

**Vercel** — `vercel.json`:
```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Content-Security-Policy", "value": "default-src 'self'; frame-ancestors 'none'" }
      ]
    }
  ]
}
```

### Express.js API

```js
const helmet = require('helmet');
app.use(helmet());
```

`helmet` sets most recommended headers by default.

## Verification

Test headers with:
- [securityheaders.com](https://securityheaders.com)
- `curl -I https://your-domain.com`
- Browser DevTools → Network tab → Response Headers
