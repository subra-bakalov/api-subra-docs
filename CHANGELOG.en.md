# Client changelog

[Български](CHANGELOG.md)

Changes to the Subra Products API that affect an integration. The newest entry is first. The full description is in the [guide](README.en.md).

## 2026-11-19

The approved catalog now has two levels.

`GET /export/approved-products` and `GET /api/v1/approved-products` return `"view": "summary"`. The list carries price, stock, images, and identifiers. The shared key reads the whole catalog from `/export/approved-products`. A client token on `/api/v1/approved-products` sees only that client's approved products. `description`, `leaflet`, `search_words`, `meta_title`, `meta_keywords`, and `meta_description` are on the detail resource:

```http
GET /api/v1/approved-products/{id}
GET /export/approved-products/{id}
```

The detail response is `"view": "full"`.

For repeat downloads:

- An error is now one JSON object: `ok`, `error` as text, `request_id`, `status`, and `violations` when present. `error.detail` is now the `error` field itself.
- `since_revision` returns changes after the revision from the previous response. `revision` is the upper bound. `removed` also includes a deleted product.
- `GET /api/v1/requests/{request_id}` with the caller's token shows the time, path, and status. Another client's id returns `404`.
- When the request includes `updated_since`, `removed` lists ids that are no longer in the catalog. The store deletes those. A full pull without `updated_since` returns an empty `removed`.
- Submitting again with the same `Idempotency-Key` and the same body returns the first submission and does not create a second one.
- The response includes `X-RateLimit-Limit` and `X-RateLimit-Remaining`. A `429` body is `{ "ok": false, "error", "limit", "window" }` and includes `Retry-After` and `X-Request-ID`.
- Every catalog response carries `X-Request-ID`. Quote that value when you report a failed call.
- The shared key allows 120 requests per minute and 3000 per hour. Above the limit the response is `429` with `Retry-After`. A client token uses the limit set on that token.
- `updated_since` returns products changed after the last successful pull.
- The response includes `ETag`. The next request with the same `If-None-Match` receives `304` and an empty body when the result is unchanged.
- The response uses `Cache-Control: private, max-age=45`.

If you read `description` from the list, switch to the single-product URL.

## 2026-10-29

The guide describes the current partner contract:

- Orders are read from `GET https://subra.bg/api/partner/orders` with the `orders:read` scope.
- Products have a `brand` field.
- The approved catalog accepts `stock`, `updated_since`, `cursor`, and returns `partner_tag`.
