# RaizeAPI plan

RaizeAPI (`raize_api`, prefix `rapi_`/`Rapi`/`RAPI_`, internals `rapi__`) is the API layer
for raize. It sits on top of raize_core and raize_resource and adds what a production JSON
API needs: typed requests, standard errors, an OpenAPI document, pagination and the HTTP
caching and safety headers.

## How it fits raize

| Package | What RaizeAPI uses from it |
|---|---|
| `raize_core` | router (`rz_router_add`, groups), `RzContext`, request/response, headers, query, middleware chain, CORS |
| `raize_resource` | resource defs (fields, actions, validations), data-layer vtable, `RzErrors`; RaizeAPI wraps its derived JSON API with typed routes, problem details, pagination and OpenAPI |
| `raize` (umbrella) | `RzApp` and the declaration function; RaizeAPI exposes `rapi_*(app, ...)` calls used inside it |
| `json` (registry) | strict RFC 8259 pull reader for request bodies |
| `codec` | JSON writer for responses; protobuf later for the gRPC gateway |
| `raize_auth` | API tokens, scopes, policies (optional, the app opts in) |
| `flux_orm` | data layer behind resources; reached only through raize_resource's vtable |

Rules kept from raize: no new compiler features, decorators only where code must be
generated, plain type-checked calls everywhere else, never crash on bad input. Framework
changes go to raize_web as short notes, never as edits here.

## Module layout

```
src/
  raize_api.mettle      facade: imports the public modules
  problem/              RFC 9457 problem details, error registry, 4xx/5xx helpers
  types/                API type descriptors (scalar, object, array, enum, nullable),
                        constraints, mapping from resource fields
  parse/                path, query, header and JSON body parsing into typed values,
                        validation, collected field errors
  respond/              JSON responses through codec, status helpers, Location, 201/204
  route/                API routes and groups, versioning (path and header), operation
                        metadata (id, summary, tags, params, bodies, responses)
  rest/                 REST resource routes from a raize_resource def
  query/                pagination (offset and cursor), filter, sort, sparse fields
  jsonapi/              optional JSON:API 1.1 documents, includes, errors
  openapi/              OpenAPI 3.1 + JSON Schema 2020-12 generation, docs page route
  negotiate/            Accept / Content-Type negotiation, 406/415
  cache/                ETag (strong and weak), If-None-Match / If-Match, Cache-Control
  idempotency/          Idempotency-Key store and replay
  ratelimit/            token bucket per key, RateLimit headers
  auth/                 glue to raize_auth: bearer tokens, scopes, policies (optional)
  grpc/                 later: gRPC gateway (codec protobuf + std/net HTTP/2)
  webhooks/             later: signed outbound webhooks with retries
tests/                  unit tests per module, conformance tests, fuzz targets
examples/               small APIs (tasks, versioned, paginated)
docs/                   guide, standards.md, references.md
```

## Build order

1. Problem details and API types (no dependencies beyond raize_core).
2. Request parsing and validation (json reader), responses (codec writer).
3. API routes, groups and versioning on raize_core's router; operation metadata.
4. REST resources from raize_resource defs, replacing the plain derived JSON API errors
   with problem details.
5. OpenAPI 3.1 document and docs page, checked with an OpenAPI validator.
6. Pagination, filtering, sorting, sparse fields.
7. Content negotiation, ETags and caching, CORS presets.
8. Idempotency keys, rate limiting.
9. Auth via raize_auth (tokens, scopes, policies).
10. Optional JSON:API 1.1.
11. Later: gRPC gateway, webhooks.

Each step lands with tests, fuzzing of every parser, and docs.

## Standards

RFC 9110 (HTTP semantics), RFC 9111 (caching), RFC 9457 (problem details), RFC 8259
(JSON), OpenAPI 3.1, JSON Schema 2020-12, JSON:API 1.1, IETF RateLimit headers draft,
IETF Idempotency-Key draft.

## References

Phoenix and Ash (AshJsonApi, AshGraphQL), FastAPI (OpenAPI from types), Django REST
Framework (pagination, filtering, throttling), Go's Huma (typed operations, problem
details), Rust's axum (extractors) and utoipa (OpenAPI from types). Ideas taken from each
are recorded in docs/references.md as they are used.
