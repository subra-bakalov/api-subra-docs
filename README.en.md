# Subra Products API - Integration guide

[Български](README.md)

> **Version:** 1.0  
> **Last updated:** April 2026

---

## Contents

1. [Quick start](#1-quick-start)
2. [Postman setup](#2-postman-setup)
3. [Authentication](#3-authentication)
4. [Workflow](#4-workflow)
5. [Categories](#5-categories)
6. [API endpoints](#6-api-endpoints)
   - [Create a product](#61-create-a-product)
   - [Update a product](#62-update-a-product)
   - [Delete a product](#63-delete-a-product)
   - [Get a product](#64-get-a-product)
   - [List products](#65-list-products)
   - [Bulk operations](#66-bulk-operations)
   - [Review submissions](#67-review-submissions)
   - [Approved catalog](#68-approved-catalog)
   - [Administrative endpoints](#69-administrative-endpoints)
7. [Product fields](#7-product-fields)
8. [Error handling](#8-error-handling)
9. [Limits and rules](#9-limits-and-rules)
10. [cURL examples](#10-curl-examples)
11. [FAQ](#11-faq)
12. [Support](#12-support)

---

## 1. Quick start

### 1.1 What the Subra Products API is

A REST API for adding and managing products in the Subra system. The API supports:

- Creating, reading, updating, and deleting products (CRUD)
- A list and status of the current client's own products
- Review submissions and status counts
- Bulk create and update
- Fast price and stock updates
- An export of the approved catalog
- Idempotent creates

### 1.2 Base URLs

| Service | URL |
|---------|-----|
| Main API | `https://api.subra.bg` |
| Categories | `https://id.subra.bg/categories` |

### 1.3 Tokens

An administrator issues every token. Each token carries specific permissions, such as `GET:/api/v1/products` or `POST:/api/v1/product-submissions`. A missing permission returns `403`.

- `main_api_token` — products and submissions on `api.subra.bg`
- `approved_products_key` — only `GET /api/v1/approved-products` and `GET /export/approved-products`

Categories at `https://id.subra.bg/categories` are public and do not require a token. There is no separate sandbox: requests go to these production URLs. A new product stays `pending` until it is approved.

---

## 2. Postman setup

### 2.1 Install

1. Install [Postman](https://www.postman.com/downloads/)
2. Download these files from this repository:
   - `Subra-Products-API.postman_collection.json`
   - `Subra-Products-API.postman_environment.json`

### 2.2 Import

1. **Import** → select both `.json` files
2. Choose the **"Subra API Environment"** environment from the dropdown in the top-right corner
3. Click the eye icon (👁️) and set:
   - `main_api_token` — your product token
   - `approved_products_key` — the approved-catalog key, if you use that feed

### 2.3 Requests in the collection

| # | Request | Description |
|---|---------|-------------|
| 1 | **Get Categories** | Fetches the category list |
| 2 | **Get Category By Id** | Details for one category |
| 3 | **List Products** | Product list with pagination |
| 4 | **Create Product With Full Payload** | Creates a product with every field |
| 5 | **Negative Test - Invalid Category** | Test with an invalid category |

> Postman stores `category_id` and `category_name` from request 1 and uses them in request 4.

---

## 3. Authentication

Every request needs a bearer token in the Authorization header:

```http
Authorization: Bearer <your_token>
```

---

## 4. Workflow

### 4.1 Creating a new product

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  1. GET         │────▶│  2. POST         │────▶│  3. An admin    │
│  /categories    │     │  /api/v1/products│     │  approves it    │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                           │
                                                           ▼
                                                 ┌─────────────────┐
                                                 │  The product    │
                                                 │  is visible in  │
                                                 │  the store      │
                                                 └─────────────────┘
```

### 4.2 Daily price and stock updates

```
┌──────────────────────────────────────────────────────────────┐
│  POST /api/v1/products/bulk/stock-price                      │
│  Updates price, oldPrice, quantity, inPromotion, visibility  │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Categories

### 5.1 Get all categories

```http
GET https://id.subra.bg/categories
```

**Example response:**

```json
[
  { "id": 3, "name": "Болкоуспокояващи" },
  { "id": 35, "name": "Витамини и минерали" },
  { "id": 87, "name": "Козметика" }
]
```

Category names in the live catalog are returned in Bulgarian.

### 5.2 Get one category

```http
GET https://id.subra.bg/categories?id={category_id}
```

A product can be assigned only to a leaf category (`isLeaf: true`). If `categoryId` or any value in `categoryIds` is a parent category, the request returns **400**.

---

## 6. API endpoints

### 6.1 Create a product

```http
POST /api/v1/products
Content-Type: application/json
Authorization: Bearer <main_api_token>
Idempotency-Key: c3b6d2f1-7b5a-4f8b-9d2a-1f2e3d4c5b6a  # Optional
```

**Example body:**

```json
{
  "title": "Advanced Retinol Serum/ Серум за красива кожа х 30 ml",
  "brand": "Example Brand",
  "categoryId": 35,
  "categoryIds": [35, 87],
  "currencyId": 1,
  "price": "8.66",
  "oldPrice": "12.90",
  "quantity": 56,
  "barcode": "3800123456789",
  "babh": "BG202600123",
  "slug": "advanced-retinol-serum-30ml",
  "text": "Висококачествен серум за красива кожа с ретинол.",
  "leaflet": "Нанасяйте вечер на чисто лице.",
  "searchWords": "ретинол серум кожа грижа",
  "metaTitle": "Advanced Retinol Serum - 30ml",
  "metaKeywords": "ретинол, серум, кожа, грижа",
  "metaDescription": "Висококачествен серум с ретинол за красива кожа.",
  "i18nForeignKey": 1001,
  "i18nLocale": "bg-BG",
  "itemWeight": 0.150,
  "itemHeight": 12.5,
  "itemWidth": 4.2,
  "itemLength": 4.2,
  "itemVolume": 0.22,
  "itemWeightUnit": "kg",
  "itemVolumeUnit": "l",
  "isNew": true,
  "isBestseller": false,
  "isByPrescription": false,
  "inPromotion": true,
  "visibility": 1,
  "activationDate": "2025-03-24T10:00:00+00:00",
  "image1": "https://example.com/product-1.jpg",
  "image2": "https://example.com/product-2.jpg"
}
```

**Success response (201):**

```json
{
  "id": 12345,
  "title": "Advanced Retinol Serum...",
  "categoryId": 35,
  "price": "8.66",
  "status": "pending",
  "submittedBy": {
    "id": 42,
    "email": "client@example.com",
    "company": "Client Company Ltd."
  }
}
```

### 6.2 Update a product

```http
PATCH /api/v1/products/{id}
Content-Type: application/json
Authorization: Bearer <main_api_token>
```

**Example body (only the fields that change):**

```json
{
  "price": "9.99",
  "quantity": 100,
  "inPromotion": true
}
```

**Alternative (POST with a method override):**

```http
POST /api/v1/products/{id}
Content-Type: multipart/form-data
X-HTTP-Method-Override: PATCH
# or _method=PATCH as a form field
```

### 6.3 Delete a product

```http
DELETE /api/v1/products/{id}
Authorization: Bearer <main_api_token>
```

### 6.4 Get a product

```http
GET /api/v1/products/{id}
Authorization: Bearer <main_api_token>
Accept: application/ld+json
```

The response includes the product and its `status`. A client can update or delete only a product that the same client submitted.

### 6.5 List products

```http
GET /api/v1/products?page=1&itemsPerPage=30
Authorization: Bearer <main_api_token>
Accept: application/ld+json
```

The list contains only products submitted by the token's user. Products are in `hydra:member`, and the total count is in `hydra:totalItems`. Every product includes `status`.

**Query parameters:**

- `page` — page number, starting at 1
- `itemsPerPage` — page size (default 30)

### 6.6 Bulk operations

#### 6.6.1 Bulk create

```http
POST /api/v1/products/bulk
Content-Type: application/json
Authorization: Bearer <main_api_token>
```

```json
{
  "items": [
    {
      "title": "Продукт 1",
      "brand": "Example Brand",
      "categoryId": 35,
      "price": "10.00",
      "quantity": 50
    },
    {
      "title": "Продукт 2",
      "categoryId": 87,
      "price": "25.00",
      "barcode": "3800123456790"
    }
  ]
}
```

**Response:**

```json
{
  "created": 2,
  "failed": 0,
  "items": [
    { "success": true, "id": 12346, "title": "Продукт 1" },
    { "success": true, "id": 12347, "title": "Продукт 2" }
  ]
}
```

> **Important:** Partial success is supported. If one product fails, the others are still created.

#### 6.6.2 Bulk update

```http
PATCH /api/v1/products/bulk
Content-Type: application/json
Authorization: Bearer <main_api_token>
```

```json
{
  "items": [
    {
      "id": 12346,
      "price": "11.00",
      "quantity": 60
    },
    {
      "id": 12347,
      "brand": "Example Brand",
      "price": "26.00",
      "inPromotion": true
    }
  ]
}
```

#### 6.6.3 Bulk price and stock update

```http
POST /api/v1/products/bulk/stock-price
Content-Type: application/json
Authorization: Bearer <main_api_token>
```

```json
{
  "effectiveAt": "2025-04-01T00:00:00+00:00",
  "items": [
    {
      "id": 12346,
      "price": "9.99",
      "oldPrice": "12.99",
      "quantity": 100,
      "inPromotion": true,
      "visibility": 1
    },
    {
      "id": 12347,
      "price": "29.99",
      "quantity": 0,
      "visibility": 0
    }
  ]
}
```

**Supported fields:** `id`, `price`, `oldPrice`, `quantity`, `inPromotion`, `visibility`, `currencyId`. A product is identified only by the `id` returned by the API.

### 6.7 Review submissions

These endpoints are not in the OpenAPI document at `/api/v1/docs`. The token needs permission for the exact path, for example `GET:/api/v1/product-submissions`. The list and the stats contain only the current client's submissions.

#### 6.7.1 Create a submission

```http
POST /api/v1/product-submissions
Content-Type: application/json
Authorization: Bearer <main_api_token>
```

```json
{
  "product": {
    "title": "Test product",
    "barcode": "3800123456789",
    "price": "10.00",
    "quantity": 50,
    "categoryId": 35
  }
}
```

If a product with the same `barcode` already exists, the submission updates that product. If that product already has a pending submission, the response is `409`.

**Success response (201):**

```json
{
  "message": "Product submitted for review",
  "submission": {
    "id": 15,
    "status": "pending",
    "submittedAt": "2026-10-26 12:00:00",
    "productId": 12345,
    "title": "Test product",
    "barcode": "3800123456789",
    "babh": null,
    "isUpdate": false
  }
}
```

#### 6.7.2 List the client's submissions

```http
GET /api/v1/product-submissions
Authorization: Bearer <main_api_token>
```

```json
{
  "submissions": [
    {
      "id": 15,
      "status": "changes_requested",
      "submittedAt": "2026-10-26 12:00:00",
      "reviewedAt": "2026-10-26 13:00:00",
      "rejectionReason": null,
      "changeRequest": "Add a leaflet.",
      "product": {
        "id": 12345,
        "title": "Test product",
        "barcode": "3800123456789",
        "babh": null,
        "price": "10.00",
        "quantity": 50,
        "visibility": 1
      },
      "changes": null,
      "isUpdate": false
    }
  ]
}
```

`rejectionReason` is set when a submission is rejected. `changeRequest` is set when an administrator asks for a correction. `changes` describes the differences when the submission updates an existing product.

#### 6.7.3 Stats

```http
GET /api/v1/product-submissions/stats
Authorization: Bearer <main_api_token>
```

```json
{
  "stats": {
    "pending": 2,
    "approved": 10,
    "rejected": 1,
    "changes_requested": 1,
    "total": 14
  }
}
```

### 6.8 Approved catalog

This exports approved products. It uses its own key, not the client CRUD token. Send the key as `X-Approved-Products-Key`, `X-Api-Key`, `Authorization: Bearer`, or the `key` query parameter.

```http
GET /api/v1/approved-products?limit=100&stock=in_stock
X-Approved-Products-Key: <approved_products_key>
```

`GET /export/approved-products` returns the same response.

**Query parameters:**

- `limit` — from 1 to 500, default 100
- `cursor` — `id` of the last product from the previous page; the next request uses `next_cursor`
- `updated_since` — ISO 8601 timestamp
- `category_id` — filter by category
- `include_hidden=1` — include products whose `visibility` is 0
- `stock` — `all`, `in_stock`, or `out_of_stock`

```json
{
  "ok": true,
  "items": [],
  "limit": 100,
  "cursor": 0,
  "next_cursor": null,
  "has_more": false,
  "filters": {
    "status": "approved",
    "updated_since": null,
    "category_id": null,
    "include_hidden": false,
    "stock": "in_stock"
  }
}
```

Each item includes `id`, `title`, `slug`, `barcode`, `price`, `old_price`, `quantity`, `visibility`, `category_id`, `category_ids`, `description`, `images`, `flags`, `created_at`, and `updated_at`.

### 6.9 Administrative endpoints

These are not available to a client token.

```http
POST /api/v1/products/bulk-approve
POST /api/v1/products/bulk-reject
Authorization: Bearer <admin_token>
Content-Type: application/json
```

```json
{ "productIds": [12345, 12346] }
```

They approve or reject only products whose status is `pending`. The token user must have the `ROLE_ADMIN` role.

```http
GET /api/v1/admin-audit/logs?page=1&limit=20
Authorization: Bearer <admin_token>
```

`limit` is at most 100. The response is `{ "logs": [], "page": 1, "limit": 20 }`.

Services inside the network can call `GET /internal/products/approved`. It requires the `X-Internal-Products-Secret` header. Parameters are `limit` (up to 200) and `updatedSince`.

---

## 7. Product fields

### 7.1 Required fields

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Product name |
| `price` | string | Price as text, for example `"8.66"` |
| `categoryId` | integer | Id of a leaf category from `/categories` |

### 7.2 Optional fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `brand` | string | Brand, up to 255 characters | `"Example Brand"` |
| `categoryIds` | array | Additional categories | `[35, 87]` |
| `currencyId` | integer | Currency (default 1 = BGN) | `1` |
| `oldPrice` | string | Previous price | `"14.90"` |
| `quantity` | integer | Stock quantity | `56` |
| `barcode` | string | Barcode | `"3800123456789"` |
| `babh` | string | Bulgarian Food Safety Agency (BABH) number | `"BG202600123"` |
| `slug` | string | URL-friendly name | `"testov-produkt"` |
| `text` | string | Description | `"Short description..."` |
| `leaflet` | string | Patient information leaflet | `"How to take the product..."` |
| `searchWords` | string | Search keywords | `"product test api"` |
| `metaTitle` | string | Meta title | `"Test product"` |
| `metaKeywords` | string | Meta keywords | `"test, api"` |
| `metaDescription` | string | Meta description | `"Example..."` |
| `i18nForeignKey` | integer | Translation group id | `1001` |
| `i18nLocale` | string | Locale | `"bg-BG"` |
| `itemWeight` | float | Weight | `0.250` |
| `itemHeight` | float | Height | `12.5` |
| `itemWidth` | float | Width | `4.2` |
| `itemLength` | float | Length | `4.2` |
| `itemVolume` | float | Volume | `0.15` |
| `itemWeightUnit` | string | Weight unit | `"kg"` |
| `itemVolumeUnit` | string | Volume unit | `"l"` |
| `isNew` | boolean | New product | `true` |
| `isBestseller` | boolean | Bestseller | `false` |
| `isByPrescription` | boolean | Prescription only | `false` |
| `inPromotion` | boolean | On promotion | `false` |
| `visibility` | integer | 0 = hidden, 1 = visible | `1` |
| `activationDate` | string | ISO 8601 date | `"2025-03-24T10:00:00+00:00"` |
| `image1` - `image5` | string | Image URL | `"https://..."` |

---

## 8. Error handling

### 8.1 HTTP status codes

| Code | Description |
|------|-------------|
| 200 | Successful request |
| 201 | Created |
| 204 | Deleted |
| 400 | Invalid data |
| 401 | Missing or invalid token |
| 403 | Not allowed |
| 404 | Product does not exist |
| 409 | Conflict (idempotency) |
| 422 | Validation errors |
| 500 | Internal error |

### 8.2 Invalid or parent category (400)

Missing category:

```json
{
  "error": {
    "status": 400,
    "title": "Bad Request",
    "detail": "Invalid category id(s): 999999. Use GET /categories to fetch a valid category id.",
    "path": "/api/v1/products",
    "violations": [
      {
        "field": "",
        "message": "Invalid category id(s): 999999. Use GET /categories to fetch a valid category id.",
        "code": null
      }
    ]
  }
}
```

Parent category:

```json
{
  "error": {
    "status": 400,
    "title": "Bad Request",
    "detail": "categoryId \"35\" is a parent category (isLeaf: false). Products can only use a leaf category.",
    "path": "/api/v1/products",
    "violations": [
      {
        "field": "",
        "message": "categoryId \"35\" is a parent category (isLeaf: false). Products can only use a leaf category.",
        "code": null
      }
    ]
  }
}
```

### 8.3 Validation errors (422)

```json
{
  "error": {
    "status": 422,
    "title": "Unprocessable Entity",
    "detail": "Validation failed.",
    "path": "/api/v1/products",
    "violations": [
      {
        "field": "images",
        "message": "Максимум 5 снимки за продукт.",
        "code": null
      }
    ]
  }
}
```

---

## 9. Limits and rules

### 9.1 Images

- At most 5 images (`image1` through `image5`)
- Send them as URLs (`https://...`)
- File uploads are not supported

### 9.2 Prices

- Prices are **strings**, not numbers
- Correct: `"8.66"`
- Incorrect: `8.66`

### 9.3 Dates

- Format: **ISO 8601**
- Example: `"2025-03-24T10:00:00+00:00"`

### 9.4 Idempotency

Send this header when creating a product:

```http
Idempotency-Key: c3b6d2f1-7b5a-4f8b-9d2a-1f2e3d4c5b6a
```

- The same key and the same payload return the same product
- The same key and a different payload return `409 Conflict`

### 9.5 Product statuses

| Status | Description |
|--------|-------------|
| `pending` | Waiting for approval |
| `approved` | Approved, and visible when `visibility` is 1 |
| `rejected` | Rejected |
| `changes_requested` | Returned for correction |

---

## 10. cURL examples

### Create a product

```bash
curl -X POST https://api.subra.bg/api/v1/products \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "title": "Test product",
    "categoryId": 35,
    "price": "10.00",
    "quantity": 50
  }'
```

### Update a price

```bash
curl -X PATCH https://api.subra.bg/api/v1/products/12345 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{"price": "9.99", "quantity": 100}'
```

### Bulk stock update

```bash
curl -X POST https://api.subra.bg/api/v1/products/bulk/stock-price \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "items": [
      {"id": 12345, "price": "9.99", "quantity": 100},
      {"id": 12346, "price": "19.99", "quantity": 50}
    ]
  }'
```

---

## 11. FAQ

**Q: When does a product become visible in the store?**  
A: After an administrator approves it and `visibility` is `1`.

**Q: Can I upload images directly?**  
A: No. Images must already be available at an external URL. Send only the links.

**Q: How do I list my products?**  
A: `GET /api/v1/products?page=1&itemsPerPage=30`. The response contains only your products.

**Q: Can I use my own ids?**  
A: No. After creation, use the `id` returned by the API. There is no `erpId` field.

**Q: Where do I send the brand?**  
A: In the `brand` field on create, update, and bulk operations.

**Q: How do I check a product's status?**  
A: `GET /api/v1/products/{id}` or the list `GET /api/v1/products`. Read `status`. The rejection reason and the requested correction are in `GET /api/v1/product-submissions`. The counts are in `GET /api/v1/product-submissions/stats`.

**Q: What is idempotency and why do I need it?**  
A: It prevents duplicates when the network fails. The same `Idempotency-Key` with the same data always returns the same result.

---

## 12. Support

For problems or questions:

- Contact the system administrator
- GitHub: https://github.com/subra-bakalov/api-subra-docs

---

*Documentation updated: 26 October 2026*
