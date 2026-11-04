# RaizeAPI

The API layer for [RaizeWeb](https://github.com/Lex-Dellimere/RaizeWeb), written in Mettle.
Package `raize_api`, prefix `rapi_`.

Build JSON APIs on raize quickly and correctly:

- REST resource and versioned routes on raize_core's router and raize_resource
- Typed parsing and validation of path, query, headers and JSON bodies
- Errors as RFC 9457 problem details
- OpenAPI 3.1 document and docs page generated from routes and types
- Cursor and offset pagination, filtering, sorting, sparse fields, optional JSON:API 1.1
- Auth (via raize_auth), CORS, rate limits, ETags, caching, idempotency keys,
  content negotiation

Status: early setup. See [PLAN.md](PLAN.md).

## License

Apache-2.0
