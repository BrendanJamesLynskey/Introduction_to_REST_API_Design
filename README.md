# 🔗 Introduction to REST API Design

An interactive Reveal.js presentation covering REST API design — HTTP methods, resource naming, status codes, pagination, versioning, authentication, caching, HATEOAS, and Express implementation.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_REST_API_Design/)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | REST API Design overview |
| 02 | Agenda | Topics at a glance |
| 03 | What Is REST? | Fielding's constraints, resource-oriented architecture |
| 04 | HTTP Methods | GET, POST, PUT, PATCH, DELETE — semantics, idempotency, safety |
| 05 | URL Design & Resource Naming | Nouns not verbs, plurals, nesting, query params |
| 06 | Request & Response Bodies | JSON structure, envelope patterns, content negotiation |
| 07 | HTTP Status Codes | 2xx, 3xx, 4xx, 5xx — when to use each |
| 08 | Error Response Format | Problem details RFC 7807, consistent error objects |
| 09 | Pagination | Offset, cursor, keyset — trade-offs, Link headers |
| 10 | Filtering, Sorting & Field Selection | Query parameter patterns, sparse fieldsets |
| 11 | Versioning Strategies | URL path, header, query param — trade-offs |
| 12 | Authentication & API Keys | Bearer tokens, API keys, OAuth scopes |
| 13 | Rate Limiting & Throttling | Token bucket, sliding window, response headers |
| 14 | HATEOAS & Hypermedia | Links, discoverability, Richardson Maturity Model |
| 15 | Caching | ETags, Cache-Control, conditional requests, CDN patterns |
| 16 | API Documentation | OpenAPI/Swagger, auto-generation, examples |
| 17 | Express Implementation | Complete REST API with routes, controllers, validation |
| 18 | Common Anti-Patterns | Chatty APIs, over-fetching, ignoring idempotency |
| 19 | Summary & Next Steps | Key takeaways and recommended reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

Fielding, R., *Architectural Styles and the Design of Network-Based Software Architectures*, 2000 · Richardson & Ruby, *RESTful Web APIs*, O'Reilly, 2013 · RFC 7231, *HTTP/1.1 Semantics and Content* · RFC 7807, *Problem Details for HTTP APIs* · RFC 8288, *Web Linking* · RFC 8594, *The Sunset HTTP Header Field* · JSON:API Specification — jsonapi.org · Microsoft REST API Guidelines — github.com/microsoft/api-guidelines · Google API Design Guide — cloud.google.com/apis/design

## License

Educational use. Code examples provided as-is.
