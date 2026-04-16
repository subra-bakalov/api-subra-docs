# Subra Products API - Ръководство за интеграция

> **Версия:** 1.0  
> **Последна актуализация:** Април 2026

---

## Съдържание

1. [Бърз старт](#1-бърз-старт)
2. [Настройка с Postman](#2-настройка-с-postman)
3. [Удостоверяване](#3-удостоверяване)
4. [Работен процес (Workflow)](#4-работен-процес-workflow)
5. [Категории](#5-категории)
6. [API Endpoints](#6-api-endpoints)
   - [Създаване на продукт](#61-създаване-на-продукт)
   - [Обновяване на продукт](#62-обновяване-на-продукт)
   - [Изтриване на продукт](#63-изтриване-на-продукт)
   - [Получаване на продукт](#64-получаване-на-продукт)
   - [Списък с продукти](#65-списък-с-продукти)
   - [Масови операции](#66-масови-операции-bulk)
7. [Полета на продукта](#7-полета-на-продукта)
8. [Обработка на грешки](#8-обработка-на-грешки)
9. [Ограничения и правила](#9-ограничения-и-правила)
10. [Примери с cURL](#10-примери-с-curl)
11. [FAQ](#11-faq)
12. [Поддръжка](#12-поддръжка)

---

## 1. Бърз старт

### 1.1 Какво е Subra Products API

REST API за добавяне и управление на продукти в системата на Subra. API-то поддържа:
- Създаване/четене/обновяване/изтриване на продукти (CRUD)
- Масови операции (bulk create/update)
- Бързо обновяване на цени и наличности
- Идемпотентност при създаване

### 1.2 Base URLs

| Сервис | URL |
|--------|-----|
| Основно API | `https://api.subra.bg` |
| Категории | `https://id.subra.bg/categories` |

### 1.3 Токени

Необходими са два токена:
- `catalog_token` - за достъп до категориите
- `main_api_token` - за CRUD операции с продукти

> Токените се получават от администратор на системата.

---

## 2. Настройка с Postman

### 2.1 Инсталация

1. Инсталирайте [Postman](https://www.postman.com/downloads/)
2. Изтеглете файловете от този репозиторий:
   - `Subra-Products-API.postman_collection.json`
   - `Subra-Products-API.postman_environment.json`

### 2.2 Импортиране

1. **Import** → изберете двата `.json` файла
2. Изберете environment **"Subra API Environment"** от dropdown в горния десен ъгъл
3. Кликнете върху очето (👁️) и редактирайте стойностите:
   - `catalog_token` - вашият токен за категории
   - `main_api_token` - вашият токен за продукти

### 2.3 Налични заявки в колекцията

| # | Заявка | Описание |
|---|--------|----------|
| 1 | **Get Categories** | Взема списък с категории |
| 2 | **Get Category By Id** | Детайли за конкретна категория |
| 3 | **List Products** | Списък с продукти (с пагинация) |
| 4 | **Create Product With Full Payload** | Създава продукт с всички полета |
| 5 | **Negative Test - Invalid Category** | Тест с невалидна категория |

> Postman автоматично записва `category_id` и `category_name` от заявка 1 в променливи за заявка 4.

---

## 3. Удостоверяване

Всички заявки изискват Bearer Token в Authorization header:

```http
Authorization: Bearer <your_token>
```

---

## 4. Работен процес (Workflow)

### 4.1 Създаване на нов продукт

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  1. GET         │────▶│  2. POST         │────▶│  3. Админ       │
│  /categories    │     │  /api/v1/products│     │  одобрява       │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                           │
                                                           ▼
                                                 ┌─────────────────┐
                                                 │  Продуктът е    │
                                                 │  видим в        │
                                                 │  магазина       │
                                                 └─────────────────┘
```

### 4.2 Ежедневно обновяване на цени/наличности

```
┌──────────────────────────────────────────────────────────────┐
│  POST /api/v1/products/bulk/stock-price                      │
│  Обновява price, oldPrice, quantity, inPromotion, visibility │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Категории

### 5.1 Получаване на всички категории

```http
GET https://id.subra.bg/categories
Authorization: Bearer <catalog_token>
```

**Примерен отговор:**
```json
[
  { "id": 3, "name": "Болкоуспокояващи" },
  { "id": 35, "name": "Витамини и минерали" },
  { "id": 87, "name": "Козметика" }
]
```

### 5.2 Получаване на конкретна категория

```http
GET https://id.subra.bg/categories?id={category_id}
Authorization: Bearer <catalog_token>
```

---

## 6. API Endpoints

### 6.1 Създаване на продукт

```http
POST /api/v1/products
Content-Type: application/json
Authorization: Bearer <main_api_token>
Idempotency-Key: c3b6d2f1-7b5a-4f8b-9d2a-1f2e3d4c5b6a  # Опционално
```

**Примерен body:**
```json
{
  "title": "Advanced Retinol Serum/ Серум за красива кожа х 30 ml",
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

**Успешен отговор (201):**
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

### 6.2 Обновяване на продукт

```http
PATCH /api/v1/products/{id}
Content-Type: application/json
Authorization: Bearer <main_api_token>
```

**Примерен body (само полетата, които се променят):**
```json
{
  "price": "9.99",
  "quantity": 100,
  "inPromotion": true
}
```

**Алтернатива (POST с override):**
```http
POST /api/v1/products/{id}
Content-Type: multipart/form-data
X-HTTP-Method-Override: PATCH
# или _method=PATCH като form поле
```

### 6.3 Изтриване на продукт

```http
DELETE /api/v1/products/{id}
Authorization: Bearer <main_api_token>
```

### 6.4 Получаване на продукт

```http
GET /api/v1/products/{id}
Authorization: Bearer <main_api_token>
```

### 6.5 Списък с продукти

```http
GET /api/v1/products?limit=20&offset=0
Authorization: Bearer <main_api_token>
```

**Query параметри:**
- `limit` - брой резултати (по подразбиране 20)
- `offset` - отместване за пагинация

### 6.6 Масови операции (Bulk)

#### 6.6.1 Масово създаване

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

**Отговор:**
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

> **Важно:** Поддържа се partial success - ако един продукт се провали, останалите все пак се създават.

#### 6.6.2 Масово обновяване

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
      "erpId": "ERP-002",
      "price": "26.00",
      "inPromotion": true
    }
  ]
}
```

#### 6.6.3 Масово обновяване на цени и наличности

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
      "erpId": "ERP-002",
      "price": "29.99",
      "quantity": 0,
      "visibility": 0
    }
  ]
}
```

**Поддържани полета:** `id`, `erpId`, `price`, `oldPrice`, `quantity`, `inPromotion`, `visibility`, `currencyId`

---

## 7. Полета на продукта

### 7.1 Задължителни полета

| Поле | Тип | Описание |
|------|-----|----------|
| `title` | string | Име на продукта |
| `price` | string | Цена (като текст, напр. "8.66") |
| `categoryId` | integer | ID на категория от `/categories` |

### 7.2 Опционални полета

| Поле | Тип | Описание | Пример |
|------|-----|----------|--------|
| `categoryIds` | array | Допълнителни категории | `[35, 87]` |
| `currencyId` | integer | Валута (по подразбиране 1=BGN) | `1` |
| `oldPrice` | string | Стара цена | `"14.90"` |
| `quantity` | integer | Наличност | `56` |
| `barcode` | string | Баркод | `"3800123456789"` |
| `babh` | string | БАБХ номер | `"BG202600123"` |
| `slug` | string | URL-friendly име | `"testov-produkt"` |
| `text` | string | Описание | `"Кратко описание..."` |
| `leaflet` | string | Листовка | `"Информация за прием..."` |
| `searchWords` | string | Ключови думи | `"продукт тест api"` |
| `metaTitle` | string | Meta заглавие | `"Тестов продукт"` |
| `metaKeywords` | string | Meta ключови думи | `"тестов, api"` |
| `metaDescription` | string | Meta описание | `"Примерен..."` |
| `i18nForeignKey` | integer | ID за превод | `1001` |
| `i18nLocale` | string | Локал | `"bg-BG"` |
| `itemWeight` | float | Тегло | `0.250` |
| `itemHeight` | float | Височина | `12.5` |
| `itemWidth` | float | Ширина | `4.2` |
| `itemLength` | float | Дължина | `4.2` |
| `itemVolume` | float | Обем | `0.15` |
| `itemWeightUnit` | string | Мерна ед. тегло | `"kg"` |
| `itemVolumeUnit` | string | Мерна ед. обем | `"l"` |
| `isNew` | boolean | Нов продукт | `true` |
| `isBestseller` | boolean | Бестселър | `false` |
| `isByPrescription` | boolean | С рецепта | `false` |
| `inPromotion` | boolean | В промоция | `false` |
| `visibility` | integer | 0=скрит, 1=видим | `1` |
| `activationDate` | string | ISO 8601 дата | `"2025-03-24T10:00:00+00:00"` |
| `image1` - `image5` | string | URL към снимки | `"https://..."` |
| `erpId` | string | Ваш вътрешен ID | `"ERP-001"` |

---

## 8. Обработка на грешки

### 8.1 HTTP Status Codes

| Код | Описание |
|-----|----------|
| 200 | Успешна заявка |
| 201 | Успешно създаване |
| 204 | Успешно изтриване |
| 400 | Невалидни данни |
| 401 | Невалиден/липсващ токен |
| 403 | Нямате права |
| 404 | Продуктът не съществува |
| 409 | Конфликт (idempotency) |
| 422 | Валидационни грешки |
| 500 | Вътрешна грешка |

### 8.2 Примерен грешен отговор

```json
{
  "error": {
    "detail": "Invalid categoryId \"999999\". Use GET /categories to fetch a valid category id."
  }
}
```

### 8.3 Валидационни грешки (422)

```json
{
  "detail": "Validation failed",
  "errors": [
    { "field": "title", "message": "Title is required" },
    { "field": "price", "message": "Price must be a positive number" }
  ]
}
```

---

## 9. Ограничения и правила

### 9.1 Снимки
- Максимум 5 снимки (`image1` до `image5`)
- Подават се като URL адреси (https://...)
- Файлови качвания не се поддържат

### 9.2 Цени
- Цените се подават като **string**, не число
- Правилно: `"8.66"`
- Грешно: `8.66`

### 9.3 Дати
- Формат: **ISO 8601**
- Пример: `"2025-03-24T10:00:00+00:00"`

### 9.4 Идемпотентност
Добавете header при създаване на продукт:
```http
Idempotency-Key: c3b6d2f1-7b5a-4f8b-9d2a-1f2e3d4c5b6a
```

- Същият key + същи payload = връща същия продукт
- Същият key + различен payload = 409 Conflict

### 9.5 Статуси на продукти

| Статус | Описание |
|--------|----------|
| `pending` | Изчаква одобрение |
| `approved` | Одобрен и видим (ако visibility=1) |
| `rejected` | Отхвърлен |

---

## 10. Примери с cURL

### Създаване на продукт
```bash
curl -X POST https://api.subra.bg/api/v1/products \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "title": "Тестов продукт",
    "categoryId": 35,
    "price": "10.00",
    "quantity": 50
  }'
```

### Обновяване на цена
```bash
curl -X PATCH https://api.subra.bg/api/v1/products/12345 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{"price": "9.99", "quantity": 100}'
```

### Масово обновяване на наличности
```bash
curl -X POST https://api.subra.bg/api/v1/products/bulk/stock-price \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "items": [
      {"id": 12345, "price": "9.99", "quantity": 100},
      {"erpId": "ERP-002", "price": "19.99", "quantity": 50}
    ]
  }'
```

---

## 11. FAQ

**Q: Кога продуктите стават видими в магазина?**  
A: След одобрение от администратор и когато `visibility=1`.

**Q: Мога ли да качвам снимки директно?**  
A: Не, снимките трябва да са на външен URL. Подавате само линковете.

**Q: Как да получа списък с моите продукти?**  
A: `GET /api/v1/products` с пагинация (limit/offset).

**Q: Може ли да използвам собствени ID-та?**  
A: Да, чрез `erpId` полето при bulk операции.

**Q: Как да проверя статуса на продукт?**  
A: `GET /api/v1/products/{id}` - проверете полето `status`.

**Q: Какво е идемпотентност и защо ми трябва?**  
A: Предпазва от дублиране при мрежови грешки. Същият Idempotency-Key със същите данни винаги връща същия резултат.

---

## 12. Поддръжка

При проблеми или въпроси:
- Свържете се с администратора на системата
- GitHub: https://github.com/subra-bakalov/api-subra-docs

---

*Документацията е актуализирана: Април 2026*
