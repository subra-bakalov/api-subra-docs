# Промени за клиенти

[English](CHANGELOG.en.md)

Тук са промените по Subra Products API, които засягат интеграцията. Най-новото е отгоре. Пълното описание е в [ръководството](README.md).

## 2026-11-19

Одобреният каталог вече има две нива.

Списъкът `GET /api/v1/approved-products` и `GET /export/approved-products` връща `"view": "summary"`. В него са цената, наличността, снимките и идентификаторите. Полетата `description`, `leaflet`, `search_words`, `meta_title`, `meta_keywords` и `meta_description` се четат от детайла:

```http
GET /api/v1/approved-products/{id}
GET /export/approved-products/{id}
```

Детайлът връща `"view": "full"`.

За повторяемо теглене:

- `updated_since` връща продуктите, променени след последния успешен час.
- Отговорът носи `ETag`. Следващата заявка със същия `If-None-Match` получава `304` и празно тяло, когато резултатът не се е променил.
- Отговорът има `Cache-Control: private, max-age=45`.

Ако четете `description` от списъка, минете към адреса за един продукт.

## 2026-10-29

Ръководството описва текущия договор за партньори:

- Поръчките се четат от `GET https://subra.bg/api/partner/orders` с право `orders:read`.
- Продуктът има поле `brand`.
- Одобреният каталог приема `stock`, `updated_since`, `cursor` и `partner_tag`.
