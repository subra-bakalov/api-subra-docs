# Subra Products API

Този пакет съдържа готов Postman проект за работа с публичното продуктово API на Subra.

## Съдържание

- `README.md`
- `Subra-Products-API.postman_collection.json`
- `Subra-Products-API.postman_environment.json`

## Настройка

1. Импортирайте collection файла в Postman.
2. Импортирайте environment файла.
3. Изберете environment-а `Subra API Environment`.
4. Попълнете:
   - `catalog_token`
   - `main_api_token`

По подразбиране се използват:

- `https://id.subra.bg/categories` за категориите
- `https://api.subra.bg` за основното API

## Препоръчителен flow

### 1. Get Categories

Заявка:

- `1. Get Categories`

Връща списък от категории:

```json
[
  { "id": 3, "name": "Болкоуспокояващи" },
  { "id": 35, "name": "Витамини и минерали" }
]
```

След изпълнение Postman автоматично записва:

- `category_id`
- `category_name`

### 2. Get Category By Id

Заявка:

- `2. Get Category By Id`

Използва текущата стойност на `category_id` и връща избраната категория.

### 3. List Products

Заявка:

- `3. List Products`

Поддържани query параметри:

- `limit`
- `offset`

Пример:

- `GET /products?limit=20&offset=0`

### 4. Create Product With Full Payload

Заявка:

- `4. Create Product With Full Payload`

Използва автоматично попълнения `category_id` от предходната стъпка.

## Поддържани полета при `POST /api/v1/products`

Следните полета са налични в примерния payload на Postman collection-а:

| Поле | Тип | Пример |
|---|---|---|
| `title` | string | `Тестов продукт от Postman` |
| `categoryId` | integer \| null | `35` |
| `currencyId` | integer \| null | `1` |
| `price` | string \| null | `"12.50"` |
| `oldPrice` | string \| null | `"14.90"` |
| `quantity` | integer \| null | `5` |
| `visibility` | integer \| null | `1` |
| `inPromotion` | boolean | `false` |
| `isNew` | boolean | `true` |
| `isBestseller` | boolean \| null | `false` |
| `isByPrescription` | boolean \| null | `false` |
| `barcode` | string \| null | `"3800123456789"` |
| `babh` | string \| null | `"BG202600123"` |
| `slug` | string \| null | `"testov-produkt-ot-postman"` |
| `text` | string \| null | `"Кратко описание на продукта."` |
| `leaflet` | string \| null | `"Информация за прием и съхранение."` |
| `searchWords` | string \| null | `"продукт тест postman"` |
| `metaTitle` | string \| null | `"Тестов продукт"` |
| `metaKeywords` | string \| null | `"тестов продукт, postman, api"` |
| `metaDescription` | string \| null | `"Примерен продукт, създаден през API."` |
| `i18nForeignKey` | integer \| null | `1001` |
| `i18nLocale` | string \| null | `"bg-BG"` |
| `itemWeight` | number \| null | `0.250` |
| `itemHeight` | number \| null | `12.5` |
| `itemWidth` | number \| null | `4.2` |
| `itemLength` | number \| null | `4.2` |
| `itemVolume` | number \| null | `0.15` |
| `itemWeightUnit` | string \| null | `"kg"` |
| `itemVolumeUnit` | string \| null | `"l"` |
| `activationDate` | ISO 8601 datetime \| null | `"2026-03-24T10:00:00+00:00"` |
| `createdAt` | ISO 8601 datetime \| null | `"2026-03-24T10:00:00+00:00"` |
| `updatedAt` | ISO 8601 datetime \| null | `"2026-03-24T10:00:00+00:00"` |
| `image1` | URL \| null | `"https://example.com/product-1.jpg"` |
| `image2` | URL \| null | `"https://example.com/product-2.jpg"` |
| `image3` | URL \| null | `"https://example.com/product-3.jpg"` |
| `image4` | URL \| null | `"https://example.com/product-4.jpg"` |
| `image5` | URL \| null | `"https://example.com/product-5.jpg"` |

Важно:

- `categoryId` трябва да бъде взет от `GET /categories`
- изображенията се подават като URL адреси
- `submittedBy` и `status` се управляват от системата

## Примерен request body

```json
{
  "title": "Тестов продукт от Postman",
  "categoryId": 35,
  "currencyId": 1,
  "price": "12.50",
  "oldPrice": "14.90",
  "quantity": 5,
  "visibility": 1,
  "barcode": "3800123456789",
  "babh": "BG202600123",
  "slug": "testov-produkt-ot-postman",
  "text": "Кратко описание на продукта.",
  "leaflet": "Информация за прием и съхранение.",
  "searchWords": "продукт тест postman",
  "metaTitle": "Тестов продукт",
  "metaKeywords": "тестов продукт, postman, api",
  "metaDescription": "Примерен продукт, създаден през API.",
  "i18nForeignKey": 1001,
  "i18nLocale": "bg-BG",
  "itemWeight": 0.250,
  "itemHeight": 12.5,
  "itemWidth": 4.2,
  "itemLength": 4.2,
  "itemVolume": 0.15,
  "itemWeightUnit": "kg",
  "itemVolumeUnit": "l",
  "isNew": true,
  "isBestseller": false,
  "isByPrescription": false,
  "inPromotion": false,
  "activationDate": "2026-03-24T10:00:00+00:00",
  "createdAt": "2026-03-24T10:00:00+00:00",
  "updatedAt": "2026-03-24T10:00:00+00:00",
  "image1": "https://example.com/product-1.jpg",
  "image2": "https://example.com/product-2.jpg",
  "image3": "https://example.com/product-3.jpg",
  "image4": "https://example.com/product-4.jpg",
  "image5": "https://example.com/product-5.jpg"
}
```

## Negative test

Заявка:

- `5. Negative Test - Invalid Category`

Очакван резултат:

- `400 Bad Request`

Пример:

```json
{
  "error": {
    "detail": "Invalid categoryId \"999999999\". Use GET /categories to fetch a valid category id."
  }
}
```

## Налични заявки в collection-а

- `1. Get Categories`
- `2. Get Category By Id`
- `3. List Products`
- `4. Create Product With Full Payload`
- `5. Negative Test - Invalid Category`

## Поддръжка

При нужда от допълнителни примери за интеграция или разширена документация, моля свържете се с екипа на Subra.
