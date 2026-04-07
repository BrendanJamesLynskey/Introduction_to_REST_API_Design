# Introduction to REST API Design

---

## Slide 01 — Title

# Introduction to REST API Design

**Principles, Patterns & Production-Ready APIs**

HTTP methods · resources · status codes · pagination · versioning · HATEOAS

---

## Slide 02 — Agenda

### Foundations
- What is REST & Fielding's constraints
- HTTP methods & semantics
- URL design & resource naming
- Request & response bodies

### Status & Errors
- HTTP status codes in depth
- Error response format (RFC 7807)
- Pagination strategies
- Filtering, sorting & field selection

### Advanced Patterns
- Versioning strategies
- Authentication & API keys
- Rate limiting & throttling
- HATEOAS & hypermedia

### Production
- Caching & conditional requests
- API documentation & OpenAPI
- Express implementation
- Common anti-patterns

---

## Slide 03 — What Is REST?

**RE**presentational **S**tate **T**ransfer was defined by Roy Fielding in his 2000 doctoral dissertation. It is an **architectural style**, not a protocol or specification. REST describes how distributed hypermedia systems should behave when designed well.

A REST API exposes **resources** (nouns) at stable URIs and lets clients manipulate them using standard HTTP methods (verbs). The server transfers **representations** of resource state — typically JSON.

### Fielding's 6 Constraints

1. **Client–Server** — separation of concerns
2. **Stateless** — each request carries all context
3. **Cacheable** — responses declare cacheability
4. **Uniform Interface** — resource identification, self-descriptive messages, HATEOAS
5. **Layered System** — intermediaries (proxies, gateways) are transparent
6. **Code-on-Demand** (optional) — server can extend client with executable code

### Resource-Oriented Architecture

- Everything is a **resource** with a unique URI
- Resources have **representations** (JSON, XML, HTML)
- State transitions via **hypermedia links**
- Standard methods operate on all resources uniformly

### REST vs RPC vs GraphQL

- **REST** — resource-centric, HTTP-native, cacheable
- **RPC** — action-centric (`POST /getUser`), tight coupling
- **GraphQL** — query language, single endpoint, client-driven schema

REST remains the dominant style for public APIs because of simplicity, cacheability, and tooling maturity.

---

## Slide 04 — HTTP Methods

REST maps CRUD operations to HTTP verbs. Understanding **safety** (no side effects) and **idempotency** (same result if repeated) is critical for correct API design.

| Method | Semantics | Safe | Idempotent | Request Body | Example |
|--------|-----------|------|------------|-------------|---------|
| **GET** | Read a resource | Yes | Yes | No | `GET /users/42` |
| **POST** | Create a resource | No | No | Yes | `POST /users` |
| **PUT** | Replace a resource entirely | No | Yes | Yes | `PUT /users/42` |
| **PATCH** | Partial update | No | No* | Yes | `PATCH /users/42` |
| **DELETE** | Remove a resource | No | Yes | Rare | `DELETE /users/42` |

**Safe Methods:** GET and HEAD never modify server state. Clients and intermediaries can retry, cache, and prefetch them freely.

**Idempotent Methods:** PUT and DELETE produce the same result no matter how many times you call them. Essential for retry logic and fault tolerance.

**PATCH Caveat:** PATCH is not inherently idempotent. `{"op":"increment","path":"/views"}` changes state on each call. JSON Merge Patch is idempotent; JSON Patch may not be.

---

## Slide 05 — URL Design & Resource Naming

### Golden Rules

- Use **nouns**, not verbs — `/users` not `/getUsers`
- Use **plurals** — `/orders` not `/order`
- Use **kebab-case** — `/line-items` not `/lineItems`
- Hierarchical nesting for ownership — `/users/42/orders`
- Limit nesting to **2 levels** max
- Use query params for filtering, not path segments

```bash
# Good URL patterns
GET    /api/v1/users              # list users
GET    /api/v1/users/42           # single user
GET    /api/v1/users/42/orders    # user's orders
POST   /api/v1/users              # create user
DELETE /api/v1/users/42           # delete user

# Bad URL patterns
GET    /api/v1/getUser?id=42      # verb in URL
POST   /api/v1/user/create        # verb + singular
GET    /api/v1/users/42/orders/7/items/3/tags  # too deep
```

### Query Parameters

- **Filtering:** `?status=active&role=admin`
- **Sorting:** `?sort=-created_at,name`
- **Pagination:** `?page=2&per_page=25`
- **Fields:** `?fields=id,name,email`
- **Search:** `?q=john+doe`

### Sub-Resources vs Top-Level

If a resource can exist independently, give it its own top-level endpoint. Use nesting only for true ownership relationships.

---

## Slide 06 — Request & Response Bodies

### JSON Structure Conventions

- Use **camelCase** for property names
- Use ISO 8601 for dates: `"2025-03-15T10:30:00Z"`
- Use `null` for absent values, don't omit keys
- Wrap collections in a named key, not bare arrays

```json
// POST /api/v1/users — request body
{
  "firstName": "Alice",
  "lastName": "Chen",
  "email": "alice@example.com",
  "role": "editor"
}
```

### Envelope Pattern

Wrap responses in a consistent envelope with `data`, `meta`, and `errors` keys.

```json
{
  "data": [
    { "id": 1, "name": "Widget A" },
    { "id": 2, "name": "Widget B" }
  ],
  "meta": {
    "totalCount": 87,
    "page": 1,
    "perPage": 25,
    "totalPages": 4
  },
  "links": {
    "self":  "/api/v1/products?page=1",
    "next":  "/api/v1/products?page=2",
    "last":  "/api/v1/products?page=4"
  }
}
```

### Content Negotiation

Clients send `Accept: application/json`. Servers respond with `Content-Type: application/json`. Return **406 Not Acceptable** if the format is unsupported.

---

## Slide 07 — HTTP Status Codes

Use the **most specific** status code that applies. Clients rely on these for control flow, retry logic, and error handling.

| Code | Name | When to Use |
|------|------|-------------|
| **200** | OK | Successful GET, PUT, PATCH |
| **201** | Created | Successful POST (include Location header) |
| **204** | No Content | Successful DELETE (no body) |
| **301** | Moved Permanently | Resource URL changed forever |
| **304** | Not Modified | Conditional GET, cache still valid |
| **400** | Bad Request | Malformed syntax, invalid body |
| **401** | Unauthorized | Missing or invalid credentials |
| **403** | Forbidden | Authenticated but not authorised |
| **404** | Not Found | Resource does not exist |
| **409** | Conflict | Duplicate, version conflict |
| **422** | Unprocessable Entity | Validation errors |
| **429** | Too Many Requests | Rate limit exceeded |
| **500** | Internal Server Error | Unexpected server failure |
| **503** | Service Unavailable | Maintenance, overloaded |

**Anti-Pattern:** Returning `200` with `{"success": false}` defeats HTTP semantics. Proxies, caches, and monitoring tools all rely on status codes for correct behaviour.

---

## Slide 08 — Error Response Format

### RFC 7807 — Problem Details

A standard format for HTTP API error responses. Provides machine-readable error details with a consistent structure.

```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Error",
  "status": 422,
  "detail": "Request body contains invalid fields",
  "instance": "/api/v1/users",
  "errors": [
    {
      "field": "email",
      "message": "must be a valid email address",
      "value": "not-an-email"
    },
    {
      "field": "age",
      "message": "must be at least 18",
      "value": 12
    }
  ]
}
```

### Required Fields

- **type** — URI identifying the error type
- **title** — short human-readable summary
- **status** — HTTP status code (repeated for convenience)
- **detail** — human-readable explanation specific to this occurrence
- **instance** — URI reference for the specific occurrence

```javascript
// Express error handler
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  res.status(status).json({
    type: `https://api.example.com/errors/${err.code}`,
    title: err.title || 'Internal Server Error',
    status,
    detail: status === 500
      ? 'An unexpected error occurred'
      : err.message,
    instance: req.originalUrl,
    ...(err.errors && { errors: err.errors })
  });
});
```

---

## Slide 09 — Pagination

Never return unbounded collections. Choose a pagination strategy based on data characteristics and client needs.

| Strategy | Params | Pros | Cons |
|----------|--------|------|------|
| **Offset** | `?page=3&per_page=25` | Simple, jump to any page | Slow on large tables, drift on inserts |
| **Cursor** | `?cursor=eyJpZCI6NDJ9&limit=25` | Stable, no drift, fast | No random page access, opaque tokens |
| **Keyset** | `?after_id=42&limit=25` | Fast (index seek), transparent | Requires sortable unique key, forward-only |

### Link Headers (RFC 8288)

```http
HTTP/1.1 200 OK
Link: </api/v1/users?cursor=abc&limit=25>; rel="next",
      </api/v1/users?cursor=xyz&limit=25>; rel="prev"
X-Total-Count: 1482
```

### Response Body Pagination

```json
{
  "data": [ "..." ],
  "meta": {
    "totalCount": 1482,
    "nextCursor": "eyJpZCI6NjcsImNyZWF0ZWQiOiIyMDI1LTAzIn0=",
    "hasMore": true
  }
}
```

---

## Slide 10 — Filtering, Sorting & Field Selection

### Filtering

Use query parameters for simple equality. Use operator suffixes or bracket notation for richer queries.

```bash
# Simple equality
GET /api/v1/products?category=electronics&in_stock=true

# Operator suffixes
GET /api/v1/products?price_gte=10&price_lte=100

# Bracket notation (JSON:API style)
GET /api/v1/products?filter[price][gte]=10&filter[price][lte]=100
```

### Sorting

Use a `sort` parameter with comma-separated fields. Prefix with `-` for descending order.

```bash
# Sort by created_at descending, then name ascending
GET /api/v1/products?sort=-created_at,name
```

### Field Selection (Sparse Fieldsets)

```bash
# Only return id, name, and price
GET /api/v1/products?fields=id,name,price
```

### Implementation Tips

- **Whitelist** sortable and filterable fields
- Return `400` for unknown field names
- Set **default sort** and **max page size**
- Index database columns used in filters
- Document supported operators per field

---

## Slide 11 — Versioning Strategies

APIs evolve. Breaking changes need a versioning strategy so existing clients keep working while new clients get new features.

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `/api/v1/users` | Visible, simple routing, cacheable | Violates REST (URI should identify resource, not version) |
| **Custom Header** | `API-Version: 2` | Clean URLs, explicit | Hidden, harder to test in browser |
| **Accept Header** | `Accept: application/vnd.api.v2+json` | Most RESTful (content negotiation) | Complex, unfamiliar to many developers |
| **Query Param** | `/api/users?version=2` | Easy to add, easy to test | Optional params can be forgotten, cache issues |

### When to Version

- **Breaking:** removing a field, changing a type, renaming a key
- **Non-breaking:** adding a new optional field, adding a new endpoint
- Version only on breaking changes
- Support at most 2–3 versions simultaneously
- Announce deprecation with `Sunset` header

### Deprecation Headers

```http
HTTP/1.1 200 OK
Sunset: Sat, 01 Nov 2025 00:00:00 GMT
Deprecation: true
Link: </api/v2/users>; rel="successor-version"
```

---

## Slide 12 — Authentication & API Keys

### Authentication Methods

| Method | Best For |
|--------|----------|
| **API Key** | Server-to-server, simple integrations |
| **Bearer Token (JWT)** | User-facing APIs, stateless auth |
| **OAuth 2.0** | Third-party access, scoped permissions |
| **Basic Auth** | Internal tools (over HTTPS only) |

```http
# API Key (header)
GET /api/v1/data
X-API-Key: sk_live_abc123def456

# Bearer Token (JWT)
GET /api/v1/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# Basic Auth
GET /api/v1/admin
Authorization: Basic YWRtaW46cGFzc3dvcmQ=
```

### JWT Middleware in Express

```javascript
const jwt = require('jsonwebtoken');

function authenticate(req, res, next) {
  const header = req.headers.authorization;
  if (!header?.startsWith('Bearer ')) {
    return res.status(401).json({
      type: '/errors/unauthorized',
      title: 'Missing or invalid token',
      status: 401
    });
  }

  try {
    const token = header.slice(7);
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    res.status(401).json({
      type: '/errors/unauthorized',
      title: 'Token expired or invalid',
      status: 401
    });
  }
}
```

### OAuth 2.0 Scopes

Scopes limit what a token can do: `read:users`, `write:orders`, `admin`. Return **403 Forbidden** when a valid token lacks the required scope.

---

## Slide 13 — Rate Limiting & Throttling

Rate limiting protects your API from abuse, ensures fair usage, and prevents cascading failures. Every production API needs it.

| Algorithm | How It Works | Trade-off |
|-----------|-------------|-----------|
| **Fixed Window** | N requests per time window | Simple, but burst at window edges |
| **Sliding Window** | Rolling window of weighted counts | Smooth, slight complexity |
| **Token Bucket** | Tokens refill at steady rate, burst OK | Allows bursts, widely used |
| **Leaky Bucket** | Requests drain at fixed rate | Smoothest output, queues requests |

### Response Headers

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 742
X-RateLimit-Reset: 1711036800

HTTP/1.1 429 Too Many Requests
Retry-After: 30
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
```

### Express Implementation

```javascript
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    type: '/errors/rate-limit',
    title: 'Too Many Requests',
    status: 429,
    detail: 'Rate limit exceeded. Try again later.'
  }
});

app.use('/api/', apiLimiter);
```

---

## Slide 14 — HATEOAS & Hypermedia

**HATEOAS** (Hypermedia As The Engine Of Application State) means the API response includes links that tell the client what actions are available next. The client never hard-codes URLs.

```json
// GET /api/v1/orders/789
{
  "data": {
    "id": 789,
    "status": "pending",
    "total": 59.99
  },
  "links": {
    "self":    { "href": "/api/v1/orders/789" },
    "cancel":  { "href": "/api/v1/orders/789/cancel", "method": "POST" },
    "pay":     { "href": "/api/v1/orders/789/pay", "method": "POST" },
    "customer": { "href": "/api/v1/users/42" }
  }
}
```

### Richardson Maturity Model

- **Level 0** — Single URI, single verb (SOAP-style RPC)
- **Level 1** — Multiple URIs (resources), but only POST
- **Level 2** — Multiple URIs + correct HTTP verbs (most APIs today)
- **Level 3** — Hypermedia controls (HATEOAS) — true REST

### Practical Reality

Most public APIs implement Level 2 only. Full HATEOAS adds complexity and many clients ignore links. GitHub, PayPal, and Spring Data REST are notable exceptions that implement HATEOAS.

---

## Slide 15 — Caching

### Cache-Control Header

```http
# Public resource — cacheable for 1 hour
Cache-Control: public, max-age=3600

# Private user data — only browser can cache
Cache-Control: private, max-age=600

# Never cache (auth endpoints, real-time data)
Cache-Control: no-store

# Cache but revalidate every time
Cache-Control: no-cache
```

### ETags & Conditional Requests

```http
# First response
HTTP/1.1 200 OK
ETag: "a1b2c3d4"

# Subsequent request — conditional
GET /api/v1/products/42
If-None-Match: "a1b2c3d4"

# Server responds — not changed
HTTP/1.1 304 Not Modified
```

### Express ETag Implementation

```javascript
app.get('/api/v1/products/:id', async (req, res) => {
  const product = await db.findById(req.params.id);
  const etag = crypto
    .createHash('md5')
    .update(JSON.stringify(product))
    .digest('hex');

  res.set({
    'ETag': `"${etag}"`,
    'Cache-Control': 'public, max-age=300',
    'Vary': 'Accept, Authorization'
  });

  if (req.headers['if-none-match'] === `"${etag}"`) {
    return res.status(304).end();
  }
  res.json({ data: product });
});
```

### CDN Patterns

Put a CDN (CloudFront, Fastly) in front of GET endpoints. Use `Vary` header to cache different representations. Invalidate on write operations.

---

## Slide 16 — API Documentation

### OpenAPI / Swagger

The **OpenAPI Specification** (OAS) is the industry standard for describing REST APIs. Write it in YAML or JSON and generate docs, SDKs, and mock servers automatically.

```yaml
openapi: 3.0.3
info:
  title: Products API
  version: 1.0.0
paths:
  /api/v1/products:
    get:
      summary: List products
      parameters:
        - name: category
          in: query
          schema:
            type: string
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: A list of products
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Product'
```

### Documentation Tools

- **Swagger UI** — interactive API explorer from OAS spec
- **Redoc** — beautiful three-panel docs from OAS
- **Postman** — collections, examples, auto-generated docs
- **Stoplight** — visual API design and documentation

### Documentation Best Practices

- Include request and response **examples** for every endpoint
- Document all error codes and their meanings
- Provide a **quick-start guide** with curl commands
- Keep docs versioned alongside code
- Add authentication instructions prominently

---

## Slide 17 — Express Implementation

A complete REST API skeleton with Express — routes, controllers, validation, and error handling.

### App Setup

```javascript
const express = require('express');
const app = express();

app.use(express.json());
app.use(cors());

app.use('/api/v1/users', require('./routes/users'));
app.use('/api/v1/products', require('./routes/products'));

// 404 handler
app.use((req, res) => {
  res.status(404).json({
    type: '/errors/not-found',
    title: 'Not Found',
    status: 404,
    detail: `No route matches ${req.method} ${req.path}`
  });
});

// Global error handler
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  res.status(status).json({
    type: `/errors/${err.code || 'server-error'}`,
    title: err.title || 'Internal Server Error',
    status,
    detail: status === 500 ? 'Something went wrong' : err.message,
    ...(err.errors && { errors: err.errors })
  });
});

app.listen(3000);
```

### Full CRUD Routes

```javascript
const router = require('express').Router();
const { body, validationResult } = require('express-validator');

// List users (paginated, filtered, sorted)
router.get('/', async (req, res, next) => {
  try {
    const { page = 1, per_page = 25, sort = '-created_at' } = req.query;
    const { data, total } = await User.findAll({
      page, perPage: per_page, sort
    });
    res.json({
      data,
      meta: { totalCount: total, page: +page, perPage: +per_page }
    });
  } catch (err) { next(err); }
});

// Create user (with validation)
router.post('/',
  body('email').isEmail(),
  body('firstName').trim().notEmpty(),
  async (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({
        type: '/errors/validation',
        title: 'Validation Error',
        status: 422,
        errors: errors.array()
      });
    }
    try {
      const user = await User.create(req.body);
      res.status(201)
        .location(`/api/v1/users/${user.id}`)
        .json({ data: user });
    } catch (err) { next(err); }
  }
);
```

---

## Slide 18 — Common Anti-Patterns

### Chatty APIs

Requiring many round-trips to accomplish a single task.

```bash
# Bad — 4 requests to load a user profile
GET /users/42
GET /users/42/address
GET /users/42/preferences
GET /users/42/avatar

# Better — compound document or expand param
GET /users/42?expand=address,preferences
```

### Over-Fetching & Under-Fetching

- **Over-fetching:** returning 50 fields when the client needs 3. Use sparse fieldsets: `?fields=id,name`
- **Under-fetching:** not enough data per response, forcing multiple calls. Provide `expand` or `include` parameters.

### Verbs in URLs

```bash
# Anti-pattern
POST /api/createUser
POST /api/deleteUser?id=42
GET  /api/getUserById?id=42

# Correct — let HTTP methods be the verbs
POST   /api/v1/users
DELETE /api/v1/users/42
GET    /api/v1/users/42
```

### Ignoring Idempotency

```javascript
// Anti-pattern: PUT creates duplicates
app.put('/orders/:id', async (req, res) => {
  await Order.create(req.body); // WRONG — always creates
});

// Correct: PUT is idempotent replace
app.put('/orders/:id', async (req, res) => {
  await Order.upsert(req.params.id, req.body);
});
```

### More Anti-Patterns

- **200 for everything** — hiding errors behind `{"error": true}`
- **Inconsistent naming** — mixing camelCase, snake_case, plural/singular
- **No pagination** — returning 100k records in one response
- **Exposing internals** — database IDs, SQL errors, stack traces
- **Breaking changes without versioning** — removing or renaming fields
- **No rate limiting** — one rogue client can take down the whole API

---

## Slide 19 — Summary & Next Steps

### Key Takeaways

- REST is an **architectural style**, not a spec
- Use **nouns** for URLs, HTTP verbs for actions
- Return correct **status codes** always
- Consistent **error format** (RFC 7807)
- Paginate, filter, and sort collections
- Version your API from day one

### Production Checklist

- Authentication & authorization
- Rate limiting on all endpoints
- Input validation & sanitisation
- Caching with ETags & Cache-Control
- Comprehensive error handling
- OpenAPI documentation
- Monitoring, logging, alerting

### Recommended Reading

- Fielding, *REST Dissertation* (2000)
- Richardson & Ruby, *RESTful Web APIs*
- RFC 7231 — HTTP Semantics
- RFC 7807 — Problem Details
- RFC 8288 — Web Linking
- JSON:API Specification
- Microsoft REST API Guidelines
- Google API Design Guide

### Next Steps

Build a complete REST API with Express. Start with 3–4 resources, implement proper status codes, validation, pagination, and error handling. Add OpenAPI docs. Deploy behind a reverse proxy with rate limiting and caching. Then explore GraphQL, gRPC, and event-driven architectures to understand when REST is — and isn't — the right choice.
