# API Design & Interoperability Cheatsheet (candidate items)

## What is interoperability?
- Two systems exchanging information AND being able to USE it
- Just exchanging bytes ≠ interoperability (need shared meaning)

## Design Principle: Shared Interfaces / Data Formats
- Both systems must agree on:
  - **Syntax** — message format (JSON, XML, etc.)
  - **Semantics** — meaning of each field

## REST APIs
- **REST** = REpresentational State Transfer
- **Stateless** protocol over HTTP / HTTPS (no server-side session)
- Each request is independent

### REST verbs (memorize)
| Verb | Action | Idempotent? |
|---|---|---|
| **GET** | Read a resource | yes |
| **POST** | Create a resource | no |
| **PUT** | Update a resource (full replace) | yes |
| **PATCH** | Update a resource (partial) | no |
| **DELETE** | Delete a resource | yes |

## URL structure
```
{protocol}://{domain}(:{port})?(/{resource})?
https://api.example.com/users/42
```

## HTTP status codes
| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 301 | Moved Permanently |
| 304 | Not Modified |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Entity |
| 429 | Too Many Requests |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

| Range | Class |
|---|---|
| 1xx | Informational |
| 2xx | Success |
| 3xx | Redirect |
| 4xx | Client error |
| 5xx | Server error |

## Alternatives to REST
- **RPC** (Remote Procedure Call) — call remote function like local
- **SOAP** (Simple Object Access Protocol) — XML-based, schema-strict
- **GraphQL** — client queries specific fields
- **gRPC** — Google's protocol-buffer-based RPC

## Schemas (interface contracts)
- **JSON Schema** — validates JSON
- **XSD** — XML schema definition
- **YAML** schemas
- **Protocol Buffers / .proto** (gRPC)
- **OpenAPI** / Swagger — REST spec

## Interface descriptions: Syntactic vs Semantic
| | Syntactic | Semantic |
|---|---|---|
| What | Parameters, types | Meaning, side effects, restrictions, errors |
| Example | `transfer(from: int, to: int, amount: float)` | "Moves amount from `from` to `to`; both must exist; amount > 0; on failure: 403 if unauthorized" |

## Documentation should include
- **Purpose / meaning** of the resource / action
- **Side effects** — changes to state
- **Usage restrictions** — who can call, under what conditions
- **Error handling** — what errors can occur and why
- **Examples** — sample input/output

## Mars Climate Orbiter (1999)
- Two systems used different units (lbf vs Newtons)
- Both interfaces matched syntactically
- Semantic mismatch → spacecraft destroyed
- $327M loss
- **Lesson:** syntactic interoperability isn't enough — define semantics

## Design Principle: Define Semantics of Shared Data
- Shared dictionary of terms with explicit definitions
- Units (always specify: kg, lb, m, ft)
- Time zones (UTC vs local)
- Number formats

## Interoperability vs Changeability trade-off
- Shared interfaces force coordinated changes
- Cannot localize a change to ONE system
- Extension/changes require:
  - New version of interface
  - Or backward-compatible additions

## Extensible Interfaces
- Add new optional fields rather than renaming
- Version your APIs (`/v1/`, `/v2/`)
- Deprecate old fields with notice
- Tolerate unknown fields gracefully

## Adapter Pattern (for interoperability)
- Wrap incompatible interface to expose a target one
- Translation layer between systems
- See `design-patterns.md`

## Evaluating practical interoperability
- Run actual exchanges between systems
- Test edge cases (large data, unusual units, time zones, encoding)
- Have a feedback loop with consumers

## Common pitfalls
- Assuming JSON validates means data is correct (syntactic only)
- Forgetting units / time zones
- Inconsistent error formats across endpoints
- Breaking changes without versioning
- "Just send all the data" — leaks internal model

## REST endpoint examples
```
GET    /users         → list
GET    /users/42      → single
POST   /users         → create (body has data)
PUT    /users/42      → replace
PATCH  /users/42      → partial update
DELETE /users/42      → remove
```

## Headers commonly seen
- `Content-Type: application/json`
- `Accept: application/json`
- `Authorization: Bearer <token>`
- `User-Agent: ...`
- `X-Request-ID: ...` (correlation)

## Additional items (potentially missing)

### Idempotency definition
- An operation is **idempotent** if running it multiple times has the same effect as running it once.
- `GET`, `PUT`, `DELETE`, `HEAD`, `OPTIONS` — idempotent
- `POST`, `PATCH` — typically NOT idempotent

### Why idempotency matters
- Network failures cause retries
- Idempotent requests are safe to retry
- Use `Idempotency-Key` header for non-idempotent operations that need retry safety

### REST principles (Roy Fielding)
1. **Client-Server** — separation of concerns
2. **Stateless** — each request has all needed info
3. **Cacheable** — responses indicate cacheability
4. **Layered system** — caches/proxies are transparent
5. **Uniform interface** — standard verbs + URIs
6. **Code on demand** (optional) — server can send executable code

### API versioning strategies
| Strategy | Example |
|---|---|
| URL path | `/v1/users`, `/v2/users` |
| Query param | `/users?version=2` |
| Header | `Accept: application/vnd.myapi.v2+json` |
| Subdomain | `v1.api.example.com` |

### Backward-compatible vs breaking changes
**Backward compatible** (safe):
- Adding new optional fields
- Adding new endpoints
- Loosening validation
- Adding default values

**Breaking** (requires new version):
- Removing fields
- Renaming fields
- Changing types
- Tightening validation
- Removing endpoints
- Changing response format

### Pagination patterns
- **Offset/limit** — `?offset=20&limit=10` (simple, but slow for deep pages)
- **Cursor** — `?cursor=abc123` (efficient, stable)
- **Page number** — `?page=3&per_page=20`

### Filtering / sorting conventions
- `GET /users?status=active&sort=name&order=desc`
- `GET /users?filter[status]=active` (JSON:API style)
- `GET /search?q=keyword`

### Error response best practices
```json
{
    "error": {
        "code": "INVALID_EMAIL",
        "message": "Email format is invalid",
        "field": "email",
        "request_id": "abc123"
    }
}
```
Consistent error format across endpoints.

### Content negotiation
- `Accept: application/json` — what client wants
- `Content-Type: application/json` — what client/server sends

### Hypermedia / HATEOAS (REST extreme)
- Responses include links to related resources
- "Discover" actions by following links
- Rare in practice; complex to implement

### Common API design mistakes
- Returning HTML errors with 200 status code
- Inconsistent naming (camelCase vs snake_case mixed)
- Returning all data (no pagination)
- Allowing unbounded queries
- Missing rate limiting
- No deprecation path
- Verbs in URLs (`/getUsers` instead of `GET /users`)
- Putting auth tokens in URL params (logged!)

### GraphQL vs REST
| | REST | GraphQL |
|---|---|---|
| Endpoints | Many | One |
| Over/under-fetching | Common | Avoided (client specifies fields) |
| Caching | HTTP-level (easy) | App-level (harder) |
| Typing | Loose | Strict schema |
| Tooling | Mature | Newer but growing |

### gRPC briefly
- Uses Protocol Buffers (binary serialization)
- Bidirectional streaming
- Strict typing via `.proto` files
- Fast, low-overhead
- Less browser-friendly than REST

### OpenAPI / Swagger
- YAML or JSON schema describing your REST API
- Generates docs, client SDKs, server stubs
- Industry standard for REST API specification

### Semantic interoperability — Mars Climate Orbiter recap
- Software A used pound-seconds; Software B expected newton-seconds
- Both used the same FORMAT (numbers)
- DIFFERENT SEMANTICS
- $327M loss, spacecraft destroyed

### Defense against semantic mismatch
- Document units explicitly
- Use strong types (e.g., `Meters` type, not `double`)
- Validate at boundaries
- Reject (don't guess) on ambiguous input

### Common authentication patterns for APIs
- API Key (`X-API-Key` header)
- Bearer Token / JWT (`Authorization: Bearer ...`)
- OAuth 2.0 (delegated auth)
- mTLS (client certificate)
- Basic Auth (rarely outside internal)

### HTTP caching headers
- `Cache-Control: max-age=3600` — cache for 1hr
- `Cache-Control: no-cache` — must revalidate
- `Cache-Control: private` — don't cache in shared cache
- `ETag: "abc123"` — entity version; for conditional GET
- `If-None-Match: "abc123"` — returns 304 if unchanged
