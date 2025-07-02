# 🛡️ Comprehensive Cache Vulnerabilities Checklist (v1.0)

This checklist is designed for security researchers, penetration testers, and developers to identify and assess cache-related vulnerabilities in web applications, including **web cache poisoning**, **cache deception**, **DNS cache poisoning**, and **CDN-specific attacks**. It also provides mitigation guidance for securing caching systems. Ideal for GitHub repositories, CI/CD pipelines, or team-based audits. Fork and enhance based on your findings!


📢 **Read the deep-dive article on LinkedIn:**  
I’ve published a detailed write-up explaining real-world cache vulnerabilities, case studies, and testing techniques.  
👉 [Read on LinkedIn](https://www.linkedin.com/feed/update/urn:li:ugcPost:7346091927957557249/)


*Version: 1.0 (July 2, 2025)*  
*Glossary*:  
- *Cache Key*: Unique identifier for cached content, often based on URL, headers, or parameters.  
- *Unkeyed Inputs*: Request components (e.g., headers) not included in the cache key but processed by the server.  
- *Cache Poisoning*: Injecting malicious content into a cache to serve to users.  
- *Cache Deception*: Tricking caches into storing sensitive dynamic content as static resources.

## ✅ Initial Detection & Infrastructure Mapping

- [ ] [High Priority] Identify caching mechanisms (browser, reverse proxy, CDN, application, database).
- [ ] Examine error pages for caching indicators (e.g., CDN-specific messages like “Cloudflare Error”).
- [ ] Inspect HTTP response headers:
  - [ ] `Cache-Control`, `Expires`, `ETag`, `Last-Modified`, `Age`
  - [ ] `Vary` header to understand cache key construction
  - [ ] CDN headers (e.g., `Server-Timing: cdn-cache`, `CF-Cache-Status`, `X-Cache`, `X-Amz-Cf-Id`, `X-Fastly-Cache`)
- [ ] Test caching across regions (e.g., send requests from different IPs to check CDN PoP behavior, use `curl --resolve`).
- [ ] Use tools:
  - [ ] **Burp Suite**: Intercept and analyze HTTP traffic for caching headers.
  - [ ] **OWASP ZAP**: Automated scanning for cache misconfigurations.
  - [ ] **curl**: Manual header inspection (e.g., `curl -I https://example.com`).

## 🔍 Web Cache Deception Testing

- [ ] Append static file extensions to dynamic URLs (e.g., `.css`, `.js`, `.avif`, `.png`, `.ico`).
- [ ] [High Priority] Test if dynamic endpoints (e.g., `/user/profile.jpg`) return sensitive data.
- [ ] Verify if responses are cached (test in incognito or another browser/session).
- [ ] Test API endpoints with static extensions (e.g., `/api/user.json`).
- [ ] Check authentication endpoints (e.g., `/account`, `/dashboard`) for cached unauthorized access.

*Example*: `curl https://example.com/user/profile.jpg` to check if sensitive data is returned.

## ☠️ Web Cache Poisoning Testing

### Unkeyed Inputs
- [ ] [Critical] Test unkeyed query parameters (e.g., `?user=admin`).
- [ ] Test unkeyed ports (e.g., `Host: example.com:8080`).
- [ ] Test unkeyed headers (e.g., `X-Forwarded-For: 1.2.3.4`, `X-Original-URL: /malicious`).
- [ ] Test `Accept-Language`, `User-Agent`, or `Referer` for server-side processing.

### Injection Attacks
- [ ] Attempt cache parameter cloaking (e.g., `?param=value;malicious=attack` or `?param=value#malicious`).
- [ ] Test cache key normalization (e.g., case sensitivity: `/Path` vs. `/path`, or `//` vs. `/`).
- [ ] Perform cache key injection (e.g., injecting `?cachebust=malicious` into cache key).
- [ ] Test reflected header injection with persistent caching (e.g., `User-Agent: <script>alert(1)</script>`).

### Smuggling and Splitting
- [ ] [Critical] Test HTTP request smuggling (e.g., CL.TE, TE.CL methods).
- [ ] Test HTTP response splitting (e.g., `%0d%0a` CRLF payloads in headers).
- [ ] Test internal cache poisoning (e.g., affecting internal proxies or microservices).

*Example*: `curl -H "X-Forwarded-For: malicious.com" https://example.com` to test unkeyed header processing.

## 🌐 DNS Cache Poisoning Testing

- [ ] Check for predictable DNS transaction IDs or source ports (e.g., use `dig +short @resolver example.com`).
- [ ] Attempt race conditions to inject forged DNS responses (e.g., flood resolver with spoofed packets).
- [ ] Verify resolver validation (e.g., check for DNSSEC or response authenticity).
- [ ] Use tools:
  - [ ] **dig**: Query DNS responses (e.g., `dig @8.8.8.8 example.com`).
  - [ ] **dnsdumpster**: Map DNS infrastructure.
  - [ ] **dnsspoof**: Test spoofed DNS responses.
  - [ ] **dnscap**: Analyze DNS traffic for anomalies.

## 🛰 CDN & Proxy-Specific Testing

### Akamai
- [ ] Check `Server-Timing: cdn-cache; desc=HIT` for cache hits.
- [ ] Inject malformed headers (e.g., `X-Cache-Key: malicious`) to test edge cache logic.
- [ ] Compare authenticated vs. unauthenticated responses for cache leaks.

### Cloudflare
- [ ] Check `CF-Cache-Status` header for caching behavior.
- [ ] Verify Cache Deception Armor status (blocks common deception attacks).
	settings page in Cloudflare dashboard).
- [ ] Test uncommon extensions (e.g., `.webp`, `.avif`) for cache bypass.

### Fastly / AWS CloudFront
- [ ] Identify headers (e.g., `X-Fastly-Cache`, `X-Amz-Cf-Id`).
- [ ] Test Fastly VCL manipulation (e.g., custom VCL rules affecting cache keys).
- [ ] Test CloudFront Lambda@Edge functions for cache behavior.
- [ ] Check TTL, purging, and stale content handling.

*Example*: `curl -H "X-Cache-Key: test" https://example.com` for Akamai cache testing.

## 🧩 Potential Enhancements (Optional but Valuable)

These are advanced checks or edge cases that can improve the depth of your cache assessments:

### 🔄 Cache Key Manipulation & Header Confusion

- [ ] Test for `ETag` manipulation (e.g., weak vs. strong, reused across users).
- [ ] [Advanced] Analyze behavior of wildcard-based cache keys or surrogate keys.
- [ ] [Advanced] Check for header confusion between hop-by-hop and end-to-end headers (e.g., `Connection`, `TE`, `Keep-Alive` being cached improperly).
- [ ] Test inconsistent behavior between CDN and origin server (e.g., conflicting TTLs or headers).

### 🔐 Auth & Role Context Issues

- [ ] [Critical] Check whether authenticated vs. unauthenticated users receive the same cached response.
- [ ] Test user role switching (e.g., admin to user) and cache artifacts (e.g., partial privilege leakage).
- [ ] Test cookie-bound resources that are cached regardless of session (e.g., static-seeming JSONs).

### 🔁 Misuse of Cache-Control Directives

- [ ] Test behavior when multiple `Cache-Control` headers are returned.
- [ ] Test malformed cache directives (e.g., `Cache-Control: max-age=999999999999`) for potential abuse.
- [ ] Examine header ordering and casing impact on caching behavior.

### 🔬 Advanced Smuggling/Desync-Based Cache Abuse

- [ ] Test for cache poisoning through request smuggling (CL.TE/TE.CL).
- [ ] Test desync attacks that result in incorrect caching of split responses.
- [ ] Monitor for orphaned payloads affecting downstream cache or microservice logic.

## 🔭 New Attack Trends to Watch (2025+)

Emerging patterns in cache and edge computing that may introduce new vulnerabilities:

### 🌍 Edge Logic & Dynamic CDN Behavior

- [ ] Review behavior of **Lambda@Edge**, **Cloudflare Workers**, and **Akamai EdgeWorkers** for dynamic cache logic.
- [ ] Check if business logic or access control is implemented at the edge — and **caches responses without revalidation**.
- [ ] Assess stale content served by edge logic during failover (`stale-while-revalidate`, `stale-if-error` abuse).

### 🤖 AI-Driven Adaptive Caching (Experimental)

- [ ] Investigate use of **AI-based TTL predictions** or **behavioral cache invalidation** in next-gen CDNs.
- [ ] Test if AI/CDN optimization models can be manipulated (e.g., training them to cache malicious or noisy endpoints).

### 🔧 Serverless / API Gateway Caching

- [ ] Test caching behavior of **API Gateway integrations** (e.g., AWS API Gateway + CloudFront).
- [ ] Identify pre-cached Lambda/API responses without auth enforcement.
- [ ] Monitor for response mutation based on unkeyed query/path variables.

### 🧱 Modern Frontend Caching Pitfalls

- [ ] Investigate caching of **client-side rendered pages (SPA)** in the CDN layer (e.g., stale or unprotected SSR output).
- [ ] Test cache pollution via **Service Workers** and browser caching APIs (`caches.open()`).
- [ ] Look for cache interactions with **GraphQL** APIs where query bodies are cached improperly.


## ⚖️ Impact Verification & Risk Assessment

- [ ] [Critical] Check cached content for sensitive data:
  - [ ] User details (e.g., names, emails)
  - [ ] Session tokens, cookies, or credentials
  - [ ] API keys, internal URLs, or business logic
- [ ] Test persistence across TTLs, regions, or sessions (e.g., check `Expires` header).
- [ ] Assess scalability (e.g., cache shared across users).
- [ ] Chain with vulnerabilities:
  - [ ] Cache poisoning + XSS
  - [ ] Cache deception + IDOR/CSRF
  - [ ] CPDoS (Cache Poisoned DoS)

## 🧪 Advanced Techniques & Tools

- [ ] Test XSS in reflected, cached headers (e.g., `User-Agent: <script>alert(1)</script>`).
- [ ] Poison error pages (e.g., 404s with `<script>malicious()</script>`).
- [ ] Use duplicate parameters to confuse cache keys (e.g., `?param=1&param=2`).
- [ ] Conduct timing attacks (e.g., measure cache hit/miss latency with `curl -w "%{time_total}"`).
- [ ] Tools:
  - [ ] **Param Miner**: Burp Suite extension for unkeyed input discovery.
  - [ ] **Web Cache Vulnerability Scanner**: CLI tool for cache poisoning tests.
  - [ ] **Burp Suite Pro**: Intercept and manipulate HTTP requests.
  - [ ] **OWASP ZAP**: Automated scanning for cache vulnerabilities.
  - [ ] **curl/httpx/ffuf**: Manual testing of headers and endpoints.
  - [ ] **wafw00f**: Identify WAFs that may affect caching.

## 🔒 Mitigation Checklist

- [ ] [Critical] Validate all inputs (headers, query parameters, cookies).
- [ ] Use `Vary` header for security-relevant inputs (e.g., `Vary: User-Agent, X-Forwarded-For`).
- [ ] Apply `Cache-Control: no-store` or `private` for sensitive content.
- [ ] Configure secure TTLs and enforce cache invalidation (e.g., purge APIs).
- [ ] Authenticate cache purge endpoints (e.g., restrict to admin IPs).
- [ ] Segment caching infrastructure from untrusted networks.
- [ ] Monitor cache hit/miss ratios and anomalies (e.g., via SIEM integration).

*Example*: Add `Cache-Control: no-store` to `/api/user` responses.

## 📝 Reporting Tips

- [ ] Provide reproduction steps with full HTTP requests (e.g., `curl` commands).
- [ ] Include screenshots of headers and response bodies (e.g., via Burp Suite).
- [ ] Document unauthenticated access to cached content (if applicable).
- [ ] Link issues to platform (e.g., Cloudflare, Fastly, internal proxy).
- [ ] Recommend specific mitigations (e.g., “Add `Vary: X-Forwarded-For` to prevent poisoning”).

## 📊 Findings Log

Use this table to track vulnerabilities:

| Endpoint | Vulnerability | Impact | Evidence | Status |
|----------|---------------|--------|----------|--------|
| `/user/profile.jpg` | Cache Deception | Exposed user data | Screenshot of cached response | Open |
| `/api/data` | Cache Poisoning | XSS via `User-Agent` | Burp Suite log | Reported |

## 📚 References

- OWASP: [Cache Poisoning Cheat Sheet](https://owasp.org/www-community/attacks/Cache_Poisoning)
- PortSwigger: [Web Cache Vulnerabilities](https://portswigger.net/web-security/web-cache-poisoning)
- Vaadata: [Web Cache Poisoning Best Practices](https://www.vaadata.com/blog/web-cache-poisoning-attacks-and-security-best-practices/)
- Snyk, Acunetix, Invicti, SOCRadar
- Based on: *Cache Vulnerabilities: A Comprehensive Technical Analysis*

## 🔁 Contribution

This checklist is open for contributions. Fork, enhance, or submit PRs with new tests or tools. Report issues for unclear things or any issues.

## End

By: Fazel Mohammad Ali Pour. .EMAD.

Special thanks to my bros Amoobehrooz and Snoopy.

I had this checklist for 2024. I've done some changes and updates, but there may be some new approaches out there.
